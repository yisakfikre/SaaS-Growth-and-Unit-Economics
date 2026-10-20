# Metric definitions

Every metric the six pages display, with its formula, unit, grain, filter behaviour and known limitations. All DAX below is transcribed from the shipped model — the complete export is [`../powerbi/measures.dax`](../powerbi/measures.dax).

---

## 1. The one thing to understand first

**MRR is not stored anywhere. It is a running total of a signed event ledger.**

```dax
MRR =
CALCULATE (
    SUM ( FactSubscriptionEvent[MRRChange] ),
    FILTER ( ALL ( DimDate ), DimDate[Date] <= MAX ( DimDate[Date] ) )
)
```

`FactSubscriptionEvent` carries one row per subscription movement, each with a signed `MRRChange`:

| EventType | Sign of `MRRChange` | Rows |
| --- | --- | --- |
| `New` | positive | 5,000 |
| `Expansion` | positive | 6,550 |
| `Contraction` | **negative** | 6,569 |
| `Churn` | **negative** | 1,413 |

Three consequences follow, and they govern everything below:

1. **MRR is an as-of-date snapshot, not a period sum.** It answers "what was the subscription base worth on this date", never "how much revenue was booked across this range". Page subtitles that read *"as of the end of the selected period"* say so for this reason.
2. **The waterfall reconciles by construction.** New + Expansion + Contraction + Churn = Net MRR Change is an identity on the same column, not two calculations that happen to agree.
3. **Churned MRR and Contraction MRR are already negative.** Measures that use them *add* rather than subtract. A formula below that looks like it has a sign error almost certainly does not — check the sign convention here first.

---

## 2. Period resolution — relative, never absolute

Every period-based measure anchors to the data's own latest date:

```dax
VAR CurrentPeriodEnd = MAX ( DimDate[Date] )
```

**No measure in this model calls `TODAY()` or contains a hardcoded date.** This is what lets the template keep working when it is loaded with a client's data that ends on a different day. It was verified by DAX inspection across every period measure during validation.

"Current month" measures resolve to the calendar month containing `MAX(DimDate[Date])`:

```dax
FILTER ( ALL ( DimDate ),
    YEAR ( DimDate[Date] ) = CurrentYear && MONTH ( DimDate[Date] ) = CurrentMonth )
```

Period-start values are taken from the **last day of the prior month**:

```dax
_MRR at Period Start =
VAR CurrentPeriodEnd = MAX ( DimDate[Date] )
VAR PeriodStart = DATE ( YEAR ( CurrentPeriodEnd ), MONTH ( CurrentPeriodEnd ), 1 )
RETURN CALCULATE ( [MRR], FILTER ( ALL ( DimDate ), DimDate[Date] = PeriodStart - 1 ) )
```

**Page 4 is the one exception.** Unit economics run on a *Last 12 complete months* window driven by a disconnected `Period Options` table, because CAC and payback are meaningless at a single-month grain. Its measures carry a `(P4)` suffix throughout — `Marketing Spend (P4)`, `Blended Marketing CAC (P4)`, and so on — to keep that different time basis visible in the measure name itself.

---

## 3. Revenue measures

| Measure | Formula | Unit | Grain |
| --- | --- | --- | --- |
| `MRR` | running total of `MRRChange` to the selected date | currency | as-of date |
| `ARR` | `[MRR] * 12` | currency | as-of date |
| `New MRR` | `MRRChange` where `EventType = "New"`, current month | currency, + | month |
| `Expansion MRR` | `MRRChange` where `EventType = "Expansion"`, current month | currency, + | month |
| `Contraction MRR` | `MRRChange` where `EventType = "Contraction"`, current month | currency, **−** | month |
| `Churned MRR` | `MRRChange` where `EventType = "Churn"`, current month | currency, **−** | month |
| `Net MRR Change` | all `MRRChange`, current month | currency, signed | month |
| `Gross New MRR` | `[New MRR] + [Expansion MRR]` | currency, + | month |
| `MRR Growth %` | `DIVIDE ( MRR − MRR(−1 month), MRR(−1 month) )` | ratio | month-over-month |
| `MRR YoY %` | same, 12 months back | ratio | year-over-year |

`ARR = MRR × 12` is a **run-rate annualisation of the current month**, not a sum of twelve months of revenue. It will not tie to a trailing-twelve-month revenue figure and is not meant to.

**Currency is deliberately unlabelled.** The model carries no currency dimension; format strings show `$` as a neutral placeholder. The template is currency-agnostic by design — a client loading data in another currency gets correct numbers with the wrong symbol, and changing the format string is a cosmetic fix.

---

## 4. Customer measures

```dax
Total Customers              = DISTINCTCOUNT of CustomerID where EventType = "New",  cumulative to date
Cumulative Churned Customers = DISTINCTCOUNT of CustomerID where EventType = "Churn", cumulative to date
Active Customers             = [Total Customers] - [Cumulative Churned Customers]
```

| Measure | Formula | Unit |
| --- | --- | --- |
| `New Customers` | distinct customers with a `New` event in the current month | count, month |
| `Churned Customers` | distinct customers with a `Churn` event in the current month | count, month |
| `Customer Growth %` | `DIVIDE ( Active − Active(−1 month), Active(−1 month) )` | ratio, MoM |
| `New Customer Rate` | `DIVIDE ( [New Customers], [_Active Customers at Period Start] )` | ratio, month |
| `ARPA` | `DIVIDE ( [MRR], [Active Customers] )` | currency per customer |

**`Active Customers` is a derived stock, not a counted one.** It infers who is still active from two cumulative event counts rather than reading a subscription status column. This is correct for this model — `FactSubscriptionEvent` is the authoritative lifecycle record — but it means a customer who churns and later returns is handled by the event ledger's own logic, not by a status flag.

**`ARPA` is a ratio, never summed.** It is recomputed from its own numerator and denominator at every grain. Summing ARPA across segments produces a meaningless number.

---

## 5. Retention and churn

```dax
Logo Churn Rate    = DIVIDE ( [Churned Customers], [_Active Customers at Period Start] )

GRR                = DIVIDE ( [_MRR at Period Start] + [Churned MRR] + [Contraction MRR],
                              [_MRR at Period Start] )

NRR                = DIVIDE ( [_MRR at Period Start] + [Churned MRR] + [Contraction MRR] + [Expansion MRR],
                              [_MRR at Period Start] )

Revenue Churn Rate = DIVIDE ( -[Churned MRR], [_MRR at Period Start] )
```

Churned and Contraction MRR are negative, so the additions above are reductions.

### 5.1 GRR and Revenue Churn Rate are not arithmetic inverses

They look like complements and are not. **GRR includes contraction; Revenue Churn Rate does not.**

```
GRR                = 1 − (churn + contraction) / starting MRR
Revenue Churn Rate =     (churn)               / starting MRR
```

At Dec 2025 the report shows **GRR 94.5%** and **Revenue Churn Rate 1.3%** — summing to 95.8%, not 100%. The gap is downgrade revenue. An earlier project assumption that these two summed to 100% was tested against live values and **disproven**; that finding is why Page 6's KPI strip carries four cards rather than three.

### 5.2 Logo churn vs revenue churn are different questions

`Logo Churn Rate` counts customers. `Revenue Churn Rate` weighs dollars. At Dec 2025: **2.4% of customers** left, carrying **1.3% of revenue** — the customers leaving were smaller than average. The two are never combined into a single "churn" figure anywhere in the report.

### 5.3 Cohort Retention %

```dax
Cohort Retention % =
VAR SelectedAge = SELECTEDVALUE ( CohortAge[Months Since Signup] )
VAR MaxDataDate = CALCULATE ( MAX ( DimDate[Date] ), ALL ( DimDate ) )
VAR CohortCustomers =
    FILTER ( DimCustomer, EDATE ( DimCustomer[SignupDate], SelectedAge ) <= MaxDataDate )
VAR CohortSize = COUNTROWS ( CohortCustomers )
VAR Retained =
    COUNTROWS (
        FILTER ( CohortCustomers,
            VAR TargetDate = EDATE ( DimCustomer[SignupDate], SelectedAge )
            VAR ChurnDate = CALCULATE ( MAX ( FactSubscriptionEvent[EventDate] ),
                                        FactSubscriptionEvent[EventType] = "Churn" )
            RETURN ISBLANK ( ChurnDate ) || ChurnDate > TargetDate ) )
RETURN DIVIDE ( Retained, CohortSize )
```

- **Cohort** = signup quarter (`DimCustomer[Signup Quarter]`, a calculated column).
- **Age** = whole months since signup, from the disconnected `CohortAge` table (`GENERATESERIES(0,35,1)`), displayed at every third month on the heatmap.
- **Denominator** = only customers who have *had the chance* to reach that age (`EDATE(SignupDate, age) <= MaxDataDate`). A cohort too young to reach a column contributes nothing rather than counting as churned — this is what produces the heatmap's empty upper triangle.
- **Retained** = no churn event, or a churn event strictly after the target date.

**Basis is customer count, not revenue.** A `Cohort Revenue Retention %` measure does exist in the model, but it is **not placed on any page and was not validated** — the revenue-weighted cohort view was deliberately deferred. Treat that measure as unfinished work left in the file, not as a feature.

**Verified checkpoints** (all cohorts combined): age 0 = 100.00%, **3 = 88.78%**, **6 = 81.00%**, **24 = 62.86%**, **30 = 61.05%**. These replaced four earlier documented values that failed to reproduce — see [VALIDATION.md](VALIDATION.md) §3.

### 5.4 Health before churn

```dax
_Health Score Before Churn =
AVERAGEX (
    FILTER ( FactSubscriptionEvent, FactSubscriptionEvent[EventType] = "Churn" ),
    VAR ChurnedCustomer = FactSubscriptionEvent[CustomerID]
    VAR ChurnDate = FactSubscriptionEvent[EventDate]
    VAR LastHealthDate =
        CALCULATE ( MAX ( FactCustomerHealth[Date] ),
            FactCustomerHealth[CustomerID] = ChurnedCustomer,
            FactCustomerHealth[Date] <= ChurnDate,
            ALL ( FactCustomerHealth ) )
    RETURN
        CALCULATE ( AVERAGE ( FactCustomerHealth[HealthScore] ),
            FactCustomerHealth[CustomerID] = ChurnedCustomer,
            FactCustomerHealth[Date] = LastHealthDate ) )
```

**The definition is "last snapshot strictly before the churn date", chosen deliberately over the alternative.** Averaging a customer's health across their whole tenure was tested and rejected: it answers a different question and systematically flatters customers who declined late. The page labels this **"before churn"**, not "at churn", because the two are not the same thing.

**Resolution limit:** health snapshots are captured roughly quarterly, so the matched value can be up to several months stale relative to the actual churn date. The page states this on itself.

| | Active baseline | Before churn |
| --- | --- | --- |
| Health Score | 68.88 | **54.71** |
| NPS | 36.89 | **12.91** |

---

## 6. Unit economics (Page 4)

All on the **Last 12 complete months** window.

```dax
Cost per Lead (P4)        = DIVIDE ( [Marketing Spend (P4)], [Leads (P4)] )
Blended Marketing CAC(P4) = DIVIDE ( [Marketing Spend (P4)], [_New Customers by Channel (P4)] )
Estimated LTV             = DIVIDE ( [ARPA] * [Assumed Gross Margin % Value],
                                     [_Smoothed Monthly Revenue Churn Rate] )
Estimated LTV:CAC (P4)    = DIVIDE ( [Estimated LTV], [Blended Marketing CAC (P4)] )
Marketing CAC Payback(P4) = DIVIDE ( [Blended Marketing CAC (P4)],
                                     [ARPA] * [Assumed Gross Margin % Value] )
```

with

```dax
_Smoothed Monthly Revenue Churn Rate = 1 - ( 1 - [_Trailing 3M Revenue Churn Rate (Cumulative)] ) ^ ( 1 / 3 )
```

### 6.1 CAC is marketing-only

`Marketing CAC` divides **marketing spend** by new customers. It excludes sales salaries, commissions, onboarding cost and overhead. A fully-loaded CAC would be materially higher. The page titles say "Marketing CAC" rather than "CAC" for this reason — the name is the disclosure.

### 6.2 The three assumptions are parameters, not constants

| Parameter | Default | Range | Drives |
| --- | --- | --- | --- |
| `Assumed Gross Margin %` | **80%** | 0–100%, step 5 | LTV, LTV:CAC, Payback. **Not** CAC |
| `LTV CAC Target Value` | **50×** | 10–100, step 5 | Scatter reference line, marker colours, the "below target" share |
| `Min Volume Value` | **30** customers | 0–100, step 5 | Which channels are judged at all |

They are `GENERATESERIES` what-if parameters wired through `SELECTEDVALUE` with those defaults, adjustable on the page. A new client should set all three to their own assumptions before reading anything on Page 4.

### 6.3 LTV formula limits

`LTV = ARPA × margin ÷ monthly revenue churn rate` is the standard simplification, and it inherits two well-known weaknesses:

- **It assumes churn is constant forever.** It is not — this report's own cohort curve shows churn is front-loaded and then flattens. A single smoothed rate understates early loss and overstates late survival.
- **The denominator is small, so the output is unstable.** With this dataset's low revenue churn, blended LTV:CAC computes to **35.7×**. That is not a realistic SaaS benchmark; it is what this formula returns on this data. The conventional 3× rule of thumb does not apply to a number produced this way, and the page's 50× default target reflects the formula's scale rather than any industry standard.

**Use LTV:CAC here as a relative ranking across channels — which is a defensible comparison, since every channel runs through the same formula — and not as an absolute figure.** Channel-level LTV uses each channel's own ARPA and churn; this is stated on the page.

### 6.4 Judgement rules

A channel is **not judged** — grey, excluded from the at/below-target split — when it has fewer than `Min Volume Value` new customers in the period. The `Other` bucket is never judged, because it aggregates channels that fall outside the Top-N cutoff and has no single economic meaning.

**Top-N here excludes; it does not create an "Other" total.** Power BI's native Top-N has no "Other" bucket for any visual type. The `Other` row on Page 4 is produced by the model's own `Channel Group` calculated column, not by the Top-N filter.

---

## 7. Segment and plan measures

| Measure | Formula | Note |
| --- | --- | --- |
| `_MRR by Plan` | `SUM(FactSubscription[MonthlyPrice])` for subscriptions live as of the date | **Different source from `[MRR]`** — see below |
| `_MRR Share of Total` | plan MRR ÷ MRR across all plans | re-sums within the current selection |
| `Churned MRR by Plan` | `[Churned MRR]` with `USERELATIONSHIP ( DimPlan[PlanID], FactSubscriptionEvent[PreviousPlanID] )` | the inactive-relationship mechanism |
| `Estimated LTV` by group | as §6, at the group's own ARPA | inherits every caveat in §6.3 |

**`_MRR by Plan` deliberately does not reuse `[MRR]`.** Plan attribution on the event fact is ambiguous — an event carries both `NewPlanID` and `PreviousPlanID` — so plan-level MRR is taken from the subscription fact instead, where plan membership is unambiguous:

```dax
_MRR by Plan =
VAR CurrentDate = MAX ( DimDate[Date] )
RETURN
CALCULATE (
    SUM ( FactSubscription[MonthlyPrice] ),
    FactSubscription[StartDate] <= CurrentDate,
    FactSubscription[EndDate] > CurrentDate || ISBLANK ( FactSubscription[EndDate] )
)
```

This reaches `DimPlan` through the **active** `FactSubscription → DimPlan` relationship and needs no `USERELATIONSHIP`. It is a genuinely different calculation path from `[MRR]`, and the two reconcile at the total because both describe the same live subscription base — not because one is derived from the other.

**Why `USERELATIONSHIP` is needed at all:** `FactSubscriptionEvent` carries *two* plan keys — `NewPlanID` and `PreviousPlanID` — so neither can be the active path to `DimPlan` without silently making the other invisible. Both relationships exist and are inactive; a measure states which one it means. **Any new plan-level measure written against event data must do the same**, or it will return unfiltered totals.

**Plans carry MRR but not customer counts.** Page 5's plan matrix says so in its subtitle. A customer can move between plans over their lifetime, so attributing a *customer* to a plan requires choosing a point in time — a decision the report does not make implicitly. Page 6's churn-by-plan visual shows churned **MRR** for the same reason: a plan-level churn *rate* would need an active-customers-by-plan denominator this model does not build.

---

## 8. Field parameters

| Parameter | Options | Used on |
| --- | --- | --- |
| `Group By` | Segment · Industry · Country | Page 5 |
| `Plan / Segment` | Plan · Segment | Page 2 |

These are `NAMEOF()`-based field parameters that swap the dimension on a visual. A field parameter **cannot** be used as a drill-through target field — the underlying column must be present — which constrains Page 5's drill-through design.

---

## 9. KPI card construction

Every KPI card on every page is three visuals stacked: a value, a delta, and a caption. Each needs its own measure, which is why the model carries 180 measures for roughly 53 business metrics:

| Layer | Table | Example |
| --- | --- | --- |
| Value | `_Measures` | `[MRR]` |
| Delta value | `Delta value` | `[_MRR Delta Value]` |
| Delta colour | `Color measures` | `[_MRR Delta Color]` |
| Caption | `Delta caption measures` | `[_MRR Delta Caption]` |
| Alt text | `Alt Text` | screen-reader description |

**A leading underscore marks an internal measure** — one that exists to construct a visual rather than to answer a business question. The three-visual overlay is deliberate: Card (new)'s own reference-label conditional formatting proved unreliable during the build, and this pattern trades object count for correctness.
