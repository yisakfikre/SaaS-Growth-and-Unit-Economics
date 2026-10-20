# Phase 5 — Step 1: DAX / Measure Validation (v2 — corrected, action-tagged)

> **v2 change log:** GRR's home corrected to P6. `[GRR]` and three Page 6 items confirmed Validated via live values reported back. A real duplicate-measure bug found in Page 4 (`Blended Marketing CAC` vs `Blended Marketing CAC (P4)`) — see the top of Section 2. Every remaining open row now carries exactly one action tag: **[DELETE]**, **[PASTE DAX]**, or **[CHECK BINDING]** — no more open-ended "needs re-test" language.


SaaS Growth & Unit Economics · Pre-production audit · No DAX modified — inventory and test plan only

> **Status definitions used throughout:**
> **Validated** — confirmed live in this project via a reported screenshot or number match.
> **Needs Re-test** — worked or was specified, but a known change, unconfirmed shape-decision, or new usage context means it must be checked again before sign-off.
> **Unverified** — built per SOP, but no live confirmation was ever reported back — genuinely unknown, not assumed passing.
> **Deprecated / Unused** — retired, deleted, or never assigned a home.

---

## 1. Complete Measure Inventory

### Page 1 — Executive Overview

| Measure | Home | Used On | Status |
|---|---|---|---|
| `[MRR]` | P1 | P1, P2, P5 | Validated |
| `[ARR]` | P1 | P1, P2 | Validated |
| `[MRR Growth %]` | P1 | P1, P2 | Validated |
| `[Active Customers]` | P1 | P1, P3, P5 | Validated |
| `[ARPA]` | P1 | P1, P4, P5, P6 | Validated |
| `[Customer Growth %]` | P1 | P1, P3 | Validated |
| `[Logo Churn Rate]` | P1 | P1, P3, P6 | Validated |
| `[NRR]` | P1 | P1, P6 | Validated |
| `[GRR]` | **P6** (P1 only ever shows it as text inside the Insight card, not as a home) | P1 (Insight card text), P6 (KPI card) | **Validated** — 94.5% confirmed live, matches independent hand-derivation (94.53%) |
| `[New MRR]`, `[Expansion MRR]`, `[Contraction MRR]`, `[Churned MRR]`, `[Net MRR Change]` | P1 | P1, P2, P6 (Churned MRR only) | Validated (New MRR was originally flagged "never fully confirmed" in early governance — treat as Needs Re-test if not since re-verified) |
| `_MRR by Plan`, `_MRR Change (Current Month)` | P1 | P1, P2, P5 | Validated |
| `_Executive Insight Text` | P1 | P1 | Validated |
| KPI delta triplets (MRR/ARR/Growth/Active Customers/ARPA/Customer Growth/Logo Churn/NRR) | P1 | P1 | Validated |
| `EventTypeSortOrder` (Power Query column) | P1 | P1, P2 | Validated |

### Page 2 — Revenue & Growth

| Measure | Home | Used On | Status |
|---|---|---|---|
| `Gross New MRR` | P2 | P2 | Validated |
| `_MRR Share of Total` | P2 | **P2, and confirmed on P5's V2 (MRR by Plan matrix), as originally specified**. Confirmed *not* mistakenly used on Page 6 — V5b correctly uses `[Churned MRR by Plan]` instead. | **[CHECK BINDING] — one specific test remains.** Location confirmed; the actual re-summing behavior isn't. Click one bubble/row on Page 5, add V2's five share percentages by hand — must total 100% *within that group*, not the unfiltered company-wide baseline. This was validated before only against Page 2's 2-option parameter; Page 5's 3-option parameter is untested. |
| Plan/Segment field parameter | P2 | P2 | Validated — distinct object from P5's "Group By" parameter, do not confuse |

### Page 3 — Customer Growth

| Measure | Home | Used On | Status |
|---|---|---|---|
| `[New Customers]` | P3 | P1, P3 | Validated (confirmed across three different filter windows via screenshots) |
| `[Churned Customers]` | P3 | P1, P3 | Validated |
| `[Total Customers]` | P3 | P3 | Validated |
| `_Active Customers Net Change`, `_Churned Customers (Negative)` | P3 | P3 | Validated |
| `_Customer Base Gap` | P3 | P3 | Validated |
| `New Customer Rate`, `_Active Customers at Period Start` | P3 | P3 | Validated (reconciliation to Customer Growth % confirmed via July 2024 churn-wave residual check) |
| `_Customer Insight Text` | P3 | — | **[DELETE]** — done, confirmed by user |
| `_New to Churned Customers Ratio` | P3 | — | **[DELETE]** — built specifically for the removed insight card, nothing else references it |
| `_Active Customers Net Change` | P1, P3 | **Active Customers KPI card caption ("+93 net vs prior month") on both Page 1 and Page 3** | **Incident, now resolved.** This was incorrectly grouped as a delete candidate in an earlier pass of this document — it was never insight-card-only, it's the direct source of the KPI caption on two pages. Deleted in error, both captions broke, restored via reconstruction (`Current − CALCULATE([Active Customers], DATEADD(DimDate[Date],-1,MONTH))`), confirmed working again. **Validated.** |
| Row-3 combined text measure (Logo churn rate + 12-month average) | P3 | — | **[CHECK YOUR MODEL]** — no confirmed standalone name available from the original build record. Look for a text-based measure combining the current rate with a 12-month average, likely near `_Logo Churn Rate 12M Average`. If found as a separate measure, delete it; keep `_Logo Churn Rate 12M Average` itself unless confirmed unused elsewhere. |

### Page 4 — Unit Economics

> ⚠️ **Critical, confirmed live: a duplicate CAC measure exists, and one version is wrong.**
> ```dax
> Blended Marketing CAC (P4) = DIVIDE([Marketing Spend (P4)], [_New Customers by Channel (P4)])   -- correct
> Blended Marketing CAC       = DIVIDE([Marketing Spend], [New Customers])                          -- WRONG: company-wide denominator
> ```
> The final consolidation decision was to fix the plain-named measure *in place*, not create a "(P4)" twin — that fix never happened. **[CHECK BINDING]** — open the Channel economics scatter and the CAC KPI card, confirm which measure is in their field wells. If it's the plain one, Page 4's CAC figures are currently computed against the wrong customer count. Report back which is bound; the wrong one gets deleted once confirmed unused.

| Measure | Home | Used On (exact visual) | Status |
|---|---|---|---|
| `[Marketing Spend]`, `[Leads]`, `[Lead to Customer Conversion Rate]` | P4 | **Confirmed: not bound directly to any visual — used as internal building blocks referenced inside other (P4-suffixed) wrapper measures.** | **Validated as an architecture pattern.** No further binding check needed for these three specifically. |
| `[Blended Marketing CAC]` (plain, wrong denominator via `[New Customers]`) | P4 | Confirmed as an internal building block, not directly visual-bound — **but which other measure references it is still unconfirmed.** | **[CHECK BINDING] — the one remaining priority item.** "Referenced by another measure" only tells us it isn't orphaned in the trivial sense; it doesn't tell us whether that reference is itself dead code or something that surfaces a wrong number downstream. Search the Fields pane for this exact measure name appearing inside another formula's text, or use a dependency/"view related" option if your version has one. Once identified, report which measure it is. |
| `Blended Marketing CAC (P4)` (correct denominator) | P4 | Confirmed bound to the Channel Economics scatter | Validated |
| `[Estimated LTV]` | P4 | P4, P5 | **Needs Re-test** — the 0.3 channel-variance test was specified but no result was ever reported back |
| `[Estimated LTV to CAC Ratio]`, `[Marketing CAC Payback (Months)]` | P4 | P4 | **Needs Re-test** — inherit both open questions above |
| `_New Customers by Channel` | P4 | P4 | Unverified — built per spec, no live number reported |
| `P4 Judgement Code`, `P4 Marker Color`, `P4 Text Color`, `P4 Spend Share*`, `P4 Blended CAC (All Channels)` | P4 | P4 | Unverified — same |
| `_P4 Period Start/End Date` | P4 | P4 | Unverified — same |
| `LTV CAC Target Value`, `Min Volume Value` (parameters) | P4 | P4 | Unverified — same |

> Page 4 is frozen, but none of the above received an explicit live-confirmed report back in this project. Freezing implies the build was acceptable visually — it does not substitute for the specific Section 0.3 confirmations this inventory is flagging as still open.

### Page 5 — Segment Analysis

| Measure | Home | Used On | Status |
|---|---|---|---|
| Group By field parameter (Segment/Industry/Country) | P5 | P5 | **Validated** — confirmed live, correctly filters both V1 (scatter) and V3 (detail matrix). **[CHECK]** whether this was tested across all three options or just one — if only one, worth a quick pass on the other two before final freeze. |
| `[Active Customers]`, `[ARPA]`, `[MRR]`, `[Estimated LTV]` (reused) | P1/P4 | P5 | Validated at source; **Needs Re-test** under P5's specific Group By/drill-through contexts |
| `_MRR by Plan`, `_MRR Share of Total` (V2 plan matrix) | P2 | P5 | **Needs Re-test — see Gate 2** |
| `DimCustomer[CustomerSegment]` as drill-through field | P5 | P5 (from P2) | **Needs Re-test — Gate 2**, explicitly flagged open in project memory |

### Page 6 — Retention & Churn

| Measure | Home | Used On | Status |
|---|---|---|---|
| `[NRR]`, `[Logo Churn Rate]` (reused) | P1 | P6 | Validated |
| `[GRR]` (reused) | P1 | P6 | **Validated — 94.5% confirmed, Gate 1 narrowed, see Section 2** |
| `_GRR Delta Value/Caption/Color` | P6 | P6 | **[PASTE DAX] — live bug suspected.** Reported as showing a +0.01pp increase colored red; GRR is normal polarity, an increase should show green. Need `_GRR Delta Color`'s literal returned hex value to determine whether this is a formula bug or a conditional-formatting binding pointed at the wrong measure. |
| `Cohort Retention %` | P6 | P6 | Validated (against corrected, live-derived reference values — the original documented values 87.09/82.95/73.45/69.37 were tested and retired) |
| `DimCustomer[Signup Quarter]`, `Signup Quarter Sort` | P6 | P6 | Validated |
| `CohortAge[Months Since Signup]` | Phase 2 | P6 | Validated (field name corrected mid-build) |
| `_Health Score Before Churn`, `_NPS Before Churn` | P6 | P6 | Validated (54.71 / 12.91, adopted as canonical after the original 60.02/19.62 reference was tested and retired) |
| `_Active Customer Health Score`, `_Active Customer NPS` | P6 | P6 | Validated (68.88 / 36.89, confirmed live after a cross-table `CALCULATE` syntax bug was found and fixed) |
| `[Churned MRR]` by `ChurnReason` (V4) | P1 | P6 | **Validated** — confirmed live, top reason ~$40K, real category names present (this also closes the "is 'Other' invented" question — genuine categories confirmed) |
| `[Churned MRR by Plan]` (V5b) | Phase 3 | P6 | Validated at source; **[CHECK]** this page isn't accidentally using `_MRR Share of Total` instead — see the Page 2 row above |
| Segment-level cumulative churn (V5a) | — | P6 | **Validated** — SMB 35.4% / Mid-Market 26.0% / Enterprise 13.7% confirmed live, exact match |
| `Segment Short Label`, `Plan Short Label` | P6 | P6 | **Validated** — confirmed rendering correctly ("Enterprise" → "Ent") |

### Cross-page / foundational

| Measure | Status |
|---|---|
| `_Data Through Label` | Validated |
| `DimAcquisitionChannel` (dimension) | Validated |
| `Assumed Gross Margin %` (parameter) | Validated |
| `_Smoothed Monthly Revenue Churn Rate` | **Keep — confirmed load-bearing.** Directly in `[Estimated LTV]`'s denominator (`DIVIDE([ARPA]*[Assumed Gross Margin % Value], [_Smoothed Monthly Revenue Churn Rate])`). Cascades to Page 4's LTV KPI card, Page 4's LTV:CAC (scatter Y-axis, Channel Detail table, judgement logic), and Page 5's V3 LTV column. |
| `_Trailing 3M Revenue Churn Rate (Cumulative)` | **Keep — confirmed load-bearing**, one hop upstream of the above (referenced directly inside `_Smoothed Monthly Revenue Churn Rate`'s formula). Same cascade applies. |
| `_Monthly Revenue Churn Rate (Complete Month)` | **Deleted, confirmed safe** — no dependent measures, no visual bindings found. |
| `[Monthly Logo Churn Rate]` | Deprecated — confirmed deleted, do not reintroduce |
| `Net New MRR YTD`, `Rolling 12M Net New MRR` | Deprecated / Unused — never assigned a page, deferred |

---

## 2. The Two Open Gates

### Gate 1 — `[GRR]` vs `[Revenue Churn Rate]` — half resolved
**`[GRR]` confirmed live at 94.5%**, matching the independent hand-derivation (94.53%) closely — `[GRR]` itself is Validated.
**Still open — [PASTE DAX]:** `[Revenue Churn Rate]`'s actual formula or live value for the same month. If it also reads ~5.5% and includes Contraction MRR, the two are inverses and Page 6 keeps 3 KPI cards. If it excludes contraction, they're distinct and a 4th card is warranted.
**Resolves:** whether Page 6's KPI strip has 3 cards or 4.

### Gate 2 — Page 5 Segment-slicer / drill-through sticky state
**Question:** does the header Segment slicer and the `DimCustomer[CustomerSegment]` drill-through filter behave correctly and distinctly — specifically, does arriving via drill-through show the correct filter card in the Filters pane, and does the state clear correctly on a fresh visit rather than "sticking" from a prior session?
**Why open:** flagged in project memory as unresolved after Page 5's build; the SOP's Checkpoint 8 (explicit Filters-pane verification) was written for this exact purpose but no result was reported back.
**Resolves:** whether Page 5 can be frozen as-is.

---

## 3. Click-by-Click Validation Checklist

Work in this order — each block's result can affect how the next one is interpreted.

### 3.1 — Gate 1: Revenue Churn Rate (GRR side already confirmed)
1. Open **Model view**. In the Fields pane, locate `[Revenue Churn Rate]`.
2. Click it once, read the formula bar. Note whether the numerator subtracts Churned MRR alone or Churned + Contraction MRR.
3. Open a blank **Table** visual on a scratch page. Add `DimDate[YearMonth]`, `[GRR]`, `[Revenue Churn Rate]`.
4. Pick the same month you already confirmed GRR=94.5% for. Add both values by hand.
   - **Sum ≈ 100%** → inverses. Page 6 keeps 3 KPI cards as currently built.
   - **Sum ≠ 100%** → distinct. Add Revenue Churn Rate as a 4th KPI card, re-spacing K1–K3 to 296px each.
5. Delete the scratch table and page.

### 3.2 — Gate 2: Page 5 drill-through
1. Go to **Page 2**. Set the Plan/Segment parameter to **Segment**.
2. Right-click any segment row → confirm **Drillthrough → Segment Analysis** appears.
3. Click it. On arrival at Page 5, open the **Filters pane** → expand **Filters on this page**.
4. Confirm it shows `DimCustomer[CustomerSegment] is <the segment you clicked>` explicitly — not blank, not a different field.
5. Confirm V1 (scatter) and V3 (detail matrix) show only that one segment.
6. Switch the Group By tile to **Industry**. Confirm the CustomerSegment filter card is still present, and V1/V3 now show industries *within* that segment only.
7. Click **Back**. Confirm Page 2 returns with its own state intact.
8. **Repeat this whole sequence a second time**, from a different segment, without closing the file in between — this is the specific test for the "sticky state" concern: confirm the second arrival shows the second segment correctly, not a leftover filter from the first visit.

### 3.3 — Page 4 core measures (Section 0.3 shape check)
1. Open `[Marketing Spend]` in the formula bar. Look for any internal date restriction (`MAX(DimDate[Date])`, `FILTER(ALL(DimDate)...)`, etc.).
2. **No internal restriction found** → it's a plain sum; Page 4's Period Options window is applied correctly at another level — confirm where.
3. **Internal single-month restriction found** → confirm the page's actual displayed values are period-*summed*, not silently showing one month under a "Last 12 complete months" label. Cross-check the KPI card's number against a manual sum of 12 months in a scratch table.
4. Repeat for `[Leads]`.
5. Open `[Lead to Customer Conversion Rate]` and `[Blended Marketing CAC]`. Confirm the denominator is `_New Customers by Channel`, not `[New Customers]`.
6. Blank table: `DimAcquisitionChannel[AcquisitionChannel]`, `[Estimated LTV]`. Confirm values genuinely differ by channel (not one repeated company-wide number).

### 3.4 — Page 6 Churn Reason categories
**Closed — confirmed live, real categories present, no action needed.**

### 3.5 — Page 6 V5a segment churn measure
**Closed — confirmed live, exact match.**

### 3.6 — Spot-check reused measures in their new contexts
1. Page 5, Group By = Segment: confirm V1 shows exactly 3 bubbles with the header Segment slicer set to "All."
2. Page 5, Group By = Country: confirm V1 and V3 both render without error at your actual country count.
3. Page 5, click one bubble: confirm V2's `_MRR Share of Total` values sum to 100% by hand — not just "look about right."
4. Page 6: confirm `[Churned MRR by Plan]` (V5b) totals reconcile against `[Churned MRR]`'s company-wide total.

---

## 4. What This Establishes

Nothing in this document changes any DAX. It separates six pages' worth of measures into what's actually confirmed versus what shipped on the strength of a frozen page looking correct. The two gates block the fewest number of downstream decisions (Page 6's KPI count, Page 5's freeze) and should be resolved first; Section 3.3–3.6 can proceed in any order after that.
