# Page 6 — Retention & Churn: Implementation SOP

SaaS Growth & Unit Economics · Phase 4 · Final page · Power BI Desktop, August 2026

> **Read first.** This SOP implements the corrected Design Specification exactly. Pages 1–5 are frozen and are not modified except where explicitly noted (there are no such exceptions on this page — unlike Page 5, nothing here requires touching an earlier page). Two items are mandatory validation gates, not optional checks — do not proceed past Section 0 until both are resolved with real evidence from your model, not assumed.

---

## Section 0 — Pre-Build Dependency Check and Mandatory Validation Gates

### 0.1 — Confirm before touching anything

| Object | Status | Action |
|---|---|---|
| `[NRR]`, `[GRR]`, `[Logo Churn Rate]` | Exist, validated | Reuse unchanged |
| `[Churned MRR]`, `[Churned MRR by Plan]` (USERELATIONSHIP) | Exist, validated | Reuse unchanged |
| `CohortAge` (disconnected, `GENERATESERIES(0,35)`) | Exists since Phase 2, never used | Reuse the table; add the filter in 0.4 |
| `DimCustomer[SignupDate]` | Exists | Source column for the new Signup Quarter column — confirmed by live DAX test; `SignupMonth` does not exist in this model |
| `CohortAge[Months Since Signup]` | Exists | The cohort-age field — confirmed by live DAX test; `CohortAge[Age]` does not exist |
| `Header_Nav_Global` | Exists, used on Pages 1–5 | Reuse — Section 2 |
| Page 4's `Period Options` disconnected table | Exists, Page-4-specific | **Do not reuse here.** Page 6 uses the standard `DimDate` slider, matching Pages 1/2/3/5. |

### 0.2 — GATE 1 (mandatory): Inspect `[GRR]` and `[Revenue Churn Rate]` DAX before finalizing the KPI count

**RESOLVED during implementation: 4-card branch confirmed.** `[Revenue Churn Rate] = DIVIDE(-[Churned MRR], [_MRR at Period Start])` — churned-only, no Contraction MRR. Checked against the same validated Dec 2025 figures as `[GRR]` (94.5%, confirmed live): Revenue Churn Rate computes to 1.30%. The two sum to 95.83%, not 100% — **confirmed genuinely distinct, not inverses.** GRR captures total revenue retention including downgrades; Revenue Churn Rate captures only outright departures. Build the 4-card strip below.

1. Open Model view → Fields pane → locate `[GRR]` and `[Revenue Churn Rate]`.
2. Open each in the formula bar. Read the actual numerator.
3. Confirm: does `[GRR]`'s numerator subtract **Churned MRR only**, or **Churned MRR + Contraction MRR**? (Re-derivation against this project's own validated Dec 2025 figures found the correct answer is the latter — churned-only reproduces 98.70%, which doesn't match the documented 94.53%; churned+contraction reproduces 94.53% exactly. Confirm this against the real formula rather than trusting the re-derivation alone.)
4. Confirm the same for `[Revenue Churn Rate]`'s numerator.
5. **If both include contraction** (or both exclude it): they are exact arithmetic inverses. Proceed with the **3-card strip** (NRR, GRR, Logo Churn Rate) as specified below — do not add Revenue Churn Rate.
6. **Confirmed: this branch is correct** — see the resolution above. Build all four cards at 303px width (not 296px, an earlier estimate here before the real width was confirmed against Page 1's precedent) — see Section 4.

<div style="border-left:3px solid #DC2626;padding-left:12px;margin:16px 0;">
<b>Do not proceed to Section 5 (KPI cards) until this gate is resolved with the real formulas.</b>
</div>

### 0.3 — GATE 2 (mandatory): Validate Cohort Retention DAX against live, cross-confirmed reference values

<blockquote>
<b>Corrected during implementation.</b> The original four reference values in this gate (87.09% / 82.95% / 73.45% / 69.37%) were carried into this project's documentation as "approved" but were never independently derived from the raw dataset — they came from unverified project notes. During the actual build, two independently-reasoned formulas were tested against them: an event-based version (checks <code>FactSubscriptionEvent</code> for a real Churn row) and a subscription-window version (checks whether an active <code>FactSubscription</code> row covers the target date). <b>Both formulas converged on the same values, exactly at older ages and within 0.1pp at younger ones</b> — strong evidence the DAX is correct and the original reference values were wrong. Age=0 was also confirmed to read exactly 100.00%, ruling out an indexing error as the cause of the gap. The reference values below replace the originals; do not reintroduce 87.09/82.95/73.45/69.37 anywhere in this build.
</blockquote>

**Corrected reference values (live-confirmed, event-based formula):**

| Age (months) | Retention % |
|---|---|
| 0 | 100.00% |
| 3 | 88.78% |
| 6 | 81.00% |
| 24 | 62.86% |
| 30 | 61.05% |

**Do not populate the full heatmap until this passes.**

1. Build the **event-based** measure exactly as given in Section 6.2 below — this is now the canonical formula (see rationale after this gate).
2. Insert a temporary Table visual (delete after this gate passes). Fields: `CohortAge[Months Since Signup]`, `Cohort Retention %`. No other filters or row context.
3. Confirm Age=0 reads 100.00% — this rules out an indexing/off-by-one error and should always be checked first, since it's the cheapest and most diagnostic single value.
4. Confirm Age=3, 6, 24, and 30 match the corrected table above.
5. **If it matches:** gate passes. Spot-check whichever additional ages your heatmap's column filter (0/3/6…33) requires, then delete the temporary table and proceed.
6. **If Age=0 is not 100.00%:** stop — this indicates a genuine indexing or date-arithmetic error, not a definitional mismatch. Do not proceed past this gate.
7. **If Age=0 is correct but other ages diverge from the corrected table above:** this now indicates an actual formula problem (unlike the original gate, these reference values have been cross-confirmed by two independent methods, not assumed) — report the actual values obtained rather than adjusting the formula to match by trial and error.

**Why the event-based formula is canonical, not the subscription-window one:** both produced matching results once fully tested, but the event-based version doesn't depend on `FactSubscription`'s `StartDate`/`EndDate` bookkeeping staying gap-free across every plan-change transition — a real fragility in a dataset where "customers can have multiple sequential subscriptions from plan changes." Simpler and less dependent wins when both are equally correct.

### 0.4 — New objects required in this build

| Object | Type | Purpose |
|---|---|---|
| `DimCustomer[Signup Quarter]` | Calculated column | Groups `SignupDate` into 12 quarterly cohort rows for V1 |
| `Cohort Retention %` | Measure | V1's core value — event-based formula, validated in Gate 2 against corrected reference values |
| `_Active Customer Health Score`, `_Active Customer NPS` | Measures | V2's active-baseline comparison values — no prior reference value exists; sanity-check against the documented ~10.5–98.0 full-population range once built |
| `_Health Score Before Churn`, `_NPS Before Churn` | Measures | V2's churned-side values — **60.02 / 19.62 retired, see note below Section 6.1** |

No other new measures are required anywhere on this page.

---

## Section 1 — Page Setup

1. Right-click any page tab → **New page** (blank — do not duplicate Page 5's tab).
2. Rename `Retention & Churn`, drag to position 6 (final page).
3. Format pane → Canvas settings → Type: Custom → Width **1280**, Height **720**.
4. **Background — leave unchanged; the theme supplies `#F4F5F7`.** Do not set manually.
5. View → Page view → Fit to page.

**Checkpoint 1** — 1280×720, gray background, no manual override, nothing else on the page yet.

---

## Section 2 — Header and Navigation `REUSE FROM PAGE 5`

1. Segment Analysis → Selection pane → `Header_Nav_Global` group → Ctrl+C.
2. Retention & Churn → Ctrl+V. Pastes at identical coordinates (X=16, Y=16, W=1248, H covering the header+nav bands through Y=90).
3. "Segment Analysis" tab → Format → Button → Style → Default → Text color `#64748B`, weight Regular.
4. "Retention & Churn" tab → color `#0F1B3D`, weight Semibold, 2px underline. Move the underline rectangle beneath it.
5. Header/nav band stays flat (no shadow) per the frozen design-system amendment; content cards built in later sections keep theme shadow.
6. **Delete the Period control carried over from Page 5's group if it differs from the standard slider** — confirm it is the `DimDate` date-range slider, not Page 4's dropdown, before proceeding. (Page 5 already uses the correct slider, so this should already be right — verify rather than assume.)

**Checkpoint 2** — "Retention & Churn" bold navy with underline; other five tabs regular slate. Header/nav band has no shadow.

---

## Section 3 — Synced Date Range Slider and Filter Flyout

1. Confirm the Period slicer copied in Section 2 is the standard `DimDate[Date]` range slider (two draggable handles), anchored to the data's last date — not "Today."
2. Segment slicer: confirmed copied from Page 5, unchanged.
3. Filters flyout `REUSE FROM PAGE 5 SHELL`: copy the shell (container, Close button, bookmark mechanism). Replace its contents with three slicers: **Industry**, **Country**, **Churn Reason**.
4. Rebuild the Filters Open/Filters Closed bookmarks fresh on this page (bookmarks are page-scoped, do not carry over with the copied group). Data unchecked, Display + Current page checked.
5. **Plan and Acquisition Channel are deliberately not in this flyout.** Plan already has a dedicated visual role (V5b); Channel remains entirely outside this page's scope. View → Sync slicers → clear the Page 6 box for both, plus Customer Health if it appears as a synced option anywhere.

**Checkpoint 3** — Period slider shows the full data range by default. Flyout contains exactly Industry, Country, Churn Reason — nothing else. Opening/closing the flyout does not reset any selection.

---

## Section 4 — KPI Cards (K1–K4, confirmed 4-card strip)

**Gate 1 resolved: `[GRR]` and `[Revenue Churn Rate]` are confirmed distinct** (94.53% vs. 1.30% at Dec 2025, summing to 95.83% — not inverses). Build all four cards below at **303px width** — the same width already established for exactly this situation on Page 1's four-card Tier 1 strip (4×303 + 3×12 gutters = 1248, the full content width).

Three-visual overlay pattern per card — confirmed still the correct construction for this Power BI version (Card (new)'s reference-label conditional formatting remains documented as unreliable; separate measures are the standard workaround), matching Pages 1–5 exactly.

### 4.1 — Build K1 (NRR) in full as the reference card

1. Insert → **Card (new)**. Data value: `[NRR]`. Title off.
2. Format → General → Properties → X=**16**, Y=**102**, Width=**303**, Height=**64**.
3. Callout value: Segoe UI Semibold, 22px, `#0F172A`. Category label: 11px, `#64748B`.
4. Reuse the existing `_NRR Delta Value` / `_NRR Delta Caption` / `_NRR Delta Color` triplet if already built from an earlier page's work; if not present in the Fields pane, build following the established pattern (glyph + sign text; neutral caption "vs prior month"; hex color field, normal polarity — higher NRR is the improvement).
5. Delta value overlay: small Card (new), positioned to sit right of the value, Width=**80**, Height=**18**. Title off, Category label off, Effects off. Conditional formatting → Font color → Field value → `_NRR Delta Color`.
6. Caption overlay: small Card (new), same X, Y+18, Width=80, Height=16. 9px `#94A3B8`. No conditional formatting.
7. Group all three → rename `KPI_NRR`.

### 4.2 — K2, K3, K4

| Card | Group | X | Measure | Polarity |
|---|---|---|---|---|
| GRR | `KPI_GRR` | **331** | `[GRR]` | Normal (up = green) |
| Logo Churn Rate | `KPI_LogoChurn` | **646** | `[Logo Churn Rate]` | Inverted (up = red) — same as Pages 1/3 |
| Revenue Churn Rate | `KPI_RevChurn` | **961** | `[Revenue Churn Rate]` | Inverted (up = red) — same reasoning as Logo Churn Rate: a rising figure is the deterioration |

Same construction pattern as K1, Y=102, Width=303, Height=64 each. X positions align to grid columns 1/4/7/10, matching Page 1's proven four-card layout exactly.

**GRR's delta triplet — confirmed not to exist elsewhere in the model, built as new. Confirmed correct via live test (reported "+0.01pp shown in green" — matches spec exactly, no fix needed):**
```dax
_GRR Delta Value = 
VAR Current = [GRR]
VAR Prior = CALCULATE ( [GRR], DATEADD ( DimDate[Date], -1, MONTH ) )
VAR PPChange = Current - Prior
RETURN
IF ( PPChange >= 0, "▲ +", "▼ " ) & FORMAT ( ABS ( PPChange ), "0.0" ) & " pp"
```
```dax
_GRR Delta Caption = "vs prior month"
```
```dax
_GRR Delta Color = 
VAR Current = [GRR]
VAR Prior = CALCULATE ( [GRR], DATEADD ( DimDate[Date], -1, MONTH ) )
RETURN
IF ( Current >= Prior, "#16A34A", IF ( Current < Prior, "#DC2626", "#94A3B8" ) )
```

**Revenue Churn Rate's delta triplet — new, same pattern, inverted polarity:**
```dax
_Revenue Churn Rate Delta Value = 
VAR Current = [Revenue Churn Rate]
VAR Prior = CALCULATE ( [Revenue Churn Rate], DATEADD ( DimDate[Date], -1, MONTH ) )
VAR PPChange = Current - Prior
RETURN
IF ( PPChange >= 0, "▲ +", "▼ " ) & FORMAT ( ABS ( PPChange ), "0.0" ) & " pp"
```
```dax
_Revenue Churn Rate Delta Caption = "vs prior month"
```
```dax
_Revenue Churn Rate Delta Color = 
VAR Current = [Revenue Churn Rate]
VAR Prior = CALCULATE ( [Revenue Churn Rate], DATEADD ( DimDate[Date], -1, MONTH ) )
RETURN
IF ( Current <= Prior, "#16A34A", IF ( Current > Prior, "#DC2626", "#94A3B8" ) )
```
Inverted from GRR's color logic — a *falling* Revenue Churn Rate is the improvement, so `Current <= Prior` maps to green here, the opposite comparison direction from GRR's triplet.

Unit is "pp" (percentage points) on both, not "%" — both are already percentages, so their month-over-month movement is a point change, matching the convention already used for Customer Growth % and Logo Churn Rate elsewhere in this report.

**Dependency shared with Gate 1:** `DATEADD(DimDate[Date], -1, MONTH)` only resolves correctly if `[GRR]` and `[Revenue Churn Rate]` are genuinely `MAX`-anchored to a single month internally. `[Revenue Churn Rate]`'s formula (`DIVIDE(-[Churned MRR], [_MRR at Period Start])`) inherits its anchoring from `[Churned MRR]` and `[_MRR at Period Start]`, both already validated elsewhere in this project as `MAX`-anchored — no separate check needed here.

**Logo Churn Rate's delta triplet already exists** (used on Pages 1 and 3) — reuse `_Logo Churn Rate Delta Value/Caption/Color` unchanged; do not rebuild.

**Checkpoint 4** — Three cards (or four, per Gate 1's outcome), correct values, correct polarity coloring, deltas showing "vs prior month" consistent with Pages 1–4's convention (not Page 4's "vs prior 12 mo," which was specific to that page's period mechanism).

---

## Section 5 — V1: Cohort Retention Heatmap

*Do not build the full visual until Gate 2 (Section 0.3) has passed.*

### 5.1 — New calculated column

1. Data view → select `DimCustomer` table → Modeling ribbon → **New column**.
```dax
Signup Quarter = "Q" & QUARTER ( DimCustomer[SignupDate] ) & " " & YEAR ( DimCustomer[SignupDate] )
```
2. Confirm 12 distinct values appear, one per quarter from 2023 Q1 through the dataset's final quarter.
3. **Sort key, required — without it the matrix sorts alphabetically ("Q1" for all years, then "Q2" for all years) instead of chronologically.** New column on `DimCustomer`:
```dax
Signup Quarter Sort = YEAR ( DimCustomer[SignupDate] ) * 10 + QUARTER ( DimCustomer[SignupDate] )
```
4. Select the `Signup Quarter` column → **Column tools** ribbon (not Modeling) → **Sort by Column** → `Signup Quarter Sort`.
5. Confirm the matrix's row sort direction is **ascending** — the confirmed source design runs oldest cohort (2023 Q4) at the top down to the most recent at the bottom.

### 5.2 — Cohort Retention % measure (canonical, event-based — validated live against corrected reference values in Gate 2)

```dax
Cohort Retention % =
VAR SelectedAge = SELECTEDVALUE ( CohortAge[Months Since Signup] )
VAR MaxDataDate = CALCULATE ( MAX ( DimDate[Date] ), ALL ( DimDate ) )
VAR CohortCustomers =
    FILTER ( DimCustomer, EDATE ( DimCustomer[SignupDate], SelectedAge ) <= MaxDataDate )
VAR CohortSize = COUNTROWS ( CohortCustomers )
VAR Retained =
    COUNTROWS (
        FILTER (
            CohortCustomers,
            VAR TargetDate = EDATE ( DimCustomer[SignupDate], SelectedAge )
            VAR ChurnDate =
                CALCULATE ( MAX ( FactSubscriptionEvent[EventDate] ), FactSubscriptionEvent[EventType] = "Churn" )
            RETURN ISBLANK ( ChurnDate ) || ChurnDate > TargetDate
        )
    )
RETURN DIVIDE ( Retained, CohortSize )
```
This is the exact field-name and date-function combination that was built and live-tested — `CohortAge[Months Since Signup]` and `DimCustomer[SignupDate]` with `EDATE`, not the `Age`/`SignupMonth`/`EOMONTH` combination an earlier draft of this SOP assumed. It's confirmed to match the corrected reference table exactly at Age 0/24/30 and within 0.1pp at Age 3/6 — build this version directly rather than the month-level alternative described in an earlier revision of this document, which referenced fields that don't exist in this model.

Note: `FILTER ( DimCustomer, ... )` — not `FILTER ( ALL ( DimCustomer ), ... )` — is deliberate. It lets the matrix's own Signup Quarter row context flow through naturally; wrapping in `ALL` would silently compute every row against the entire customer base instead of just that quarter's cohort.

### 5.3 — Build the visual

1. Insert → **Matrix**. X=**16**, Y=**178**, Width=**828**, Height=**240**.
2. Rows: `DimCustomer[Signup Quarter]`.
3. Columns: `CohortAge[Months Since Signup]`.
4. Values: `Cohort Retention %`.
5. Filters pane → drag `CohortAge[Months Since Signup]` into Filters on this visual → Filter type: **Advanced filtering** → "is one of" → values 0, 3, 6, 9, 12, 15, 18, 21, 24, 27, 30, 33. No new column required for this — a pure visual-level filter on the existing disconnected table.
6. Format visual → **Cell elements** → apply Background color conditional formatting → Field value → `Cohort Retention %` → Format style: Gradient, single color `#0F1B3D` — Minimum: lowest observed value (the corrected reference values put Age-30 retention at ≈61%, and later ages will run lower still — do not hardcode a guessed number, read the actual minimum from the built matrix), Maximum: 100%. This produces the single-hue navy intensity ramp — confirm no diverging or red/green option is accidentally selected.
7. **Make cell text invisible, color-only.** At 12×12 grid density (~61×15px per cell) in-cell percentage text is not legible anyway; color alone should carry the magnitude, with exact values living in the tooltip (5.4). <b>Correction: there is no direct "hide cell text" toggle in Power BI</b> — an earlier version of this step cited one that doesn't exist. The real technique: Cell elements → add a second rule under <b>Font color</b> (Format style: Gradient) → set its Minimum and Maximum to the exact same colors as the Background color rule above. This makes the text blend into its own cell's background rather than being removed. Confirmed during implementation: if this Font color rule is skipped, cell numbers render visibly in the default font color, which also measurably increases render time (Performance Analyzer showed 2,457ms render on this visual with text visible; expect some reduction once Font color is applied, though the exact amount wasn't isolated).
8. Row/column headers: 9px `#64748B` — the one exception to the 10px content-text floor, matching the axis-furniture precedent already established on Page 5's scatter.
9. Confirm empty cells (a cohort not yet old enough to reach that age — e.g., the 2026 Q3 row has no Age-30 column) render fully transparent, not a colored zero. If the matrix shows a colored cell for a not-yet-observed combination, the underlying measure is returning 0% instead of blank for an out-of-range cohort — recheck the `CohortCustomers` filter in 5.2.
10. Title "Customer retention by cohort", 14px Semibold. Subtitle "Signup quarter × cohort age, quarterly checkpoints · hover a cell for the exact value", 10px `#94A3B8`.
11. Legend text box: "Lower — Higher retention" with a small gradient swatch, positioned top-right of the card, matching the mockup.

### 5.4 — Tooltip

Default tooltip, fields: `Signup Quarter`, `Months Since Signup` (rename display to "Cohort Age"), `Cohort Retention %`.

### 5.5 — Interactions

Format → Edit interactions → set V1's effect on every other visual on the page to **None**, in both directions. V1 answers a structurally different, longitudinal question than everything else here — cross-filtering it would misrepresent that, and no click behavior is intended.

**Checkpoint 5** — Gate 2 already passed (against the corrected reference values) before this section began. Full heatmap renders with a triangular populated region (upper-right empty/transparent). Hovering any populated cell shows exact quarter, age, and percentage. No cell text visible at rest. Selecting/clicking anywhere on this visual changes nothing elsewhere on the page.

---

## Section 6 — V2: Health Before Churn

### 6.1 — New measures

```dax
_Health Score Before Churn =
AVERAGEX (
    FILTER ( FactSubscriptionEvent, FactSubscriptionEvent[EventType] = "Churn" ),
    VAR ChurnedCustomer = FactSubscriptionEvent[CustomerID]
    VAR ChurnDate = FactSubscriptionEvent[EventDate]
    VAR LastHealthDate =
        CALCULATE (
            MAX ( FactCustomerHealth[Date] ),
            FactCustomerHealth[CustomerID] = ChurnedCustomer,
            FactCustomerHealth[Date] <= ChurnDate,
            ALL ( FactCustomerHealth )
        )
    RETURN
        CALCULATE (
            AVERAGE ( FactCustomerHealth[HealthScore] ),
            FactCustomerHealth[CustomerID] = ChurnedCustomer,
            FactCustomerHealth[Date] = LastHealthDate
        )
)
```
```dax
_NPS Before Churn =
AVERAGEX (
    FILTER ( FactSubscriptionEvent, FactSubscriptionEvent[EventType] = "Churn" ),
    VAR ChurnedCustomer = FactSubscriptionEvent[CustomerID]
    VAR ChurnDate = FactSubscriptionEvent[EventDate]
    VAR LastHealthDate =
        CALCULATE (
            MAX ( FactCustomerHealth[Date] ),
            FactCustomerHealth[CustomerID] = ChurnedCustomer,
            FactCustomerHealth[Date] <= ChurnDate,
            ALL ( FactCustomerHealth )
        )
    RETURN
        CALCULATE (
            AVERAGE ( FactCustomerHealth[NPS] ),
            FactCustomerHealth[CustomerID] = ChurnedCustomer,
            FactCustomerHealth[Date] = LastHealthDate
        )
)
```

```dax
_Active Customer Health Score =
AVERAGEX (
    FILTER (
        DimCustomer,
        NOT (
            CALCULATE (
                COUNTROWS ( FactSubscriptionEvent ),
                FactSubscriptionEvent[EventType] = "Churn"
            ) > 0
        )
    ),
    VAR ThisCustomer = DimCustomer[CustomerID]
    VAR LastHealthDate =
        CALCULATE (
            MAX ( FactCustomerHealth[Date] ),
            FactCustomerHealth[CustomerID] = ThisCustomer,
            ALL ( FactCustomerHealth )
        )
    RETURN
        CALCULATE (
            AVERAGE ( FactCustomerHealth[HealthScore] ),
            FactCustomerHealth[CustomerID] = ThisCustomer,
            FactCustomerHealth[Date] = LastHealthDate
        )
)
```
<blockquote>
<b>Corrected during implementation.</b> The original version compared <code>FactSubscriptionEvent[CustomerID] = DimCustomer[CustomerID]</code> directly as a CALCULATE filter argument — invalid, since a simple (non-FILTER-wrapped) CALCULATE condition can only reference one table's columns, and this mixed two. The churn check now relies on CALCULATE's automatic context transition through the active <code>DimCustomer → FactSubscriptionEvent</code> relationship instead, which makes the explicit condition both invalid and unnecessary. The <code>FactCustomerHealth</code> lookups now capture the customer as a scalar variable (<code>ThisCustomer</code>) rather than assuming a <code>DimCustomer → FactCustomerHealth</code> relationship is active — that path has never been confirmed elsewhere in this project, so this mirrors the same scalar-comparison technique already proven working in <code>_Health Score Before Churn</code> rather than relying on an unverified relationship.
</blockquote>
```dax
_Active Customer NPS =
AVERAGEX (
    FILTER (
        DimCustomer,
        NOT (
            CALCULATE (
                COUNTROWS ( FactSubscriptionEvent ),
                FactSubscriptionEvent[EventType] = "Churn"
            ) > 0
        )
    ),
    VAR ThisCustomer = DimCustomer[CustomerID]
    VAR LastHealthDate =
        CALCULATE (
            MAX ( FactCustomerHealth[Date] ),
            FactCustomerHealth[CustomerID] = ThisCustomer,
            ALL ( FactCustomerHealth )
        )
    RETURN
        CALCULATE (
            AVERAGE ( FactCustomerHealth[NPS] ),
            FactCustomerHealth[CustomerID] = ThisCustomer,
            FactCustomerHealth[Date] = LastHealthDate
        )
)
```

<blockquote>
<b>Corrected during implementation — 60.02 / 19.62 retired.</b> These two figures were carried into this project's documentation as approved reference values but, like the original cohort retention figures, were never independently derived from the raw dataset. Two definitions of "health before churn" were tested against them during the build: <b>last snapshot before churn</b> (this measure's construction, confirmed live at Health Score 54.71 / NPS 12.91) and <b>average across the customer's full tenure up to churn</b> (Health Score 64.25). Neither matched 60.02/19.62 — the reference sits between the two tested definitions rather than confirming either, which rules out both as the original method rather than validating one. A search for the original provenance (e.g. a Phase 3 Python computation, the same process that originally validated the cohort figures) turned up nothing. <b>Last-snapshot-before-churn is adopted as canonical</b> — it directly answers this visual's stated question ("what did this customer's health look like right before they left"), where the tenure average would instead answer a different question and systematically flatter customers who declined late in their relationship. Do not reintroduce 60.02/19.62 anywhere in this build.
</blockquote>

**Corrected reference values (live-confirmed):** `_Health Score Before Churn` = **54.71**, `_NPS Before Churn` = **12.91**. `_Active Customer Health Score` = **68.88**, `_Active Customer NPS` = **36.89** — both confirmed meaningfully higher than their churned-side counterparts and within the documented ~10.5–98.0 full-population range for Health Score.

**Validate immediately after building, before placing anything on the canvas:** confirm all four measures reproduce the corrected values above.

### 6.2 — Build the visual

1. Container: Rectangle, X=**856**, Y=**178**, Width=**408**, Height=**240**. Theme shadow, no manual border.
2. Title "Health at churn", 14px Semibold. Subtitle "Active baseline vs. last observed before churn", 10px `#94A3B8`.
3. Row 1 — "Health Score" label, 11px `#64748B`.
4. Two Card (new) visuals: `_Active Customer Health Score` (label "Active customers" above it, 9px `#64748B`) and `_Health Score Before Churn` (label "Before churn"). Values 20px Semibold — active in `#0F172A`, before-churn in `#DC2626`.
5. **Add a small down-arrow glyph (▼) immediately before the before-churn value**, same `#DC2626`. This is the accessibility fix from the design audit — color is never the sole signal on this project, and a plain red number with no other cue would violate that rule everywhere else it's enforced.
6. Small right-arrow (`→`) or connecting line between the two values, `#94A3B8`, indicating comparison direction.
7. Repeat rows 3–5 for NPS.
8. Footnote text box, 9px `#94A3B8`: "Health snapshots are captured roughly quarterly, so the matched 'before churn' value can be up to several months stale relative to the actual churn date."

**Checkpoint 6** — Before-churn values read 54.71 (Health Score) and 12.91 (NPS), per the corrected reference above. Active-customer values read 68.88 and 36.89, meaningfully higher in both cases. The ▼ glyph is present — no value on this card relies on color alone.

---

## Section 7 — V3: GRR vs NRR Trend

1. Insert → **Line chart**. X=**436**, Y=**430**, Width=**408**, Height=**274**.
2. X-axis: `DimDate[YearMonth]`, **full report period** — confirm no relative-date or 12-month filter is applied to this visual; it must show all 36 months, correcting the mockup's mistaken "Trailing 12 months" framing.
3. Y-axis: `[NRR]` (line 1) and `[GRR]` (line 2), one shared axis.
4. Lines: NRR `#0F1B3D` 2px, GRR `#94A3B8` 2px. Round joins, markers off except last point.
5. Y-axis: percentage format, auto-scaled.
6. Gridlines `#EEF0F3`, horizontal only.
7. Legend: top-right, "NRR" / "GRR".
8. Title "GRR vs. NRR", 14px Semibold. Subtitle "Full period", 10px `#94A3B8`.
9. **Do not add any annotation or callout marking specific months** — any visible widening of the gap near known soft-economy months should emerge from the real data, not from a hardcoded label.

**Checkpoint 7** — Chart spans January 2023 through the dataset's final month. Two distinct lines, GRR consistently at or below NRR (GRR excludes expansion's offsetting effect).

---

## Section 8 — V4: Churned MRR by Reason

1. **Before building:** Data view → `FactSubscriptionEvent` column `ChurnReason` → click the column, check the Fields pane's distinct value count, or drop it into a blank table visual filtered to `EventType = "Churn"`. **Record the actual category list.** If a value resembling "Other" or blank/unknown genuinely exists in the data, it is real and may be used. If no such value exists, do not add one — this project's reusability rule bans inventing an "Other" bucket that doesn't come from the data itself.
2. Insert → **Horizontal bar chart**. X=**16**, Y=**430**, Width=**408**, Height=**274**.
3. Axis: `FactSubscriptionEvent[ChurnReason]`. Values: `[Churned MRR]`.
4. Sort: descending by `[Churned MRR]` — confirmed as the primary sort; do not sort by count.
5. Bars: `#0F1B3D` throughout, single hue — this is a magnitude comparison across categories, not a judgment encoding.
6. Data labels on, showing the MRR value, 10px `#0F172A`.
7. Tooltip: add `[Churned Customers]` as a secondary field so count is visible on hover without competing with MRR as the primary visual signal.
8. Title "Churn by reason", 14px Semibold. Subtitle "Sorted by churned MRR", 10px `#94A3B8`.
9. **Do not word the title or any label as implying causation** — "reason" reflects a recorded category, not a proven cause.

**Checkpoint 8** — Bars sorted by dollar impact, not count. Every category shown is confirmed to exist in the real `ChurnReason` field from step 1 — none invented. Hovering shows customer count alongside MRR.

---

## Section 9 — V5a: Cumulative Logo Churn % by Segment

### 9.1 — New calculated column: hand-picked abbreviations, with a generic fallback

At 198px wide, full segment names ("Mid-Market," "Enterprise") don't fit as axis labels. A purely algorithmic truncation ("Ent," "Mid," first-3-letters) doesn't read as naturally as a hand-picked abbreviation — but a hand-picked one is a genuine, named exception to this template's general reusability rule, not something to introduce silently. It's accepted here on the same precedent already used elsewhere in this project: the MRR-movement waterfall's `EventTypeSortOrder` is also a hardcoded Power Query column, justified because that category *structure* is intrinsic to this domain. Plan and Segment are treated the same way — "5 named plans," "3 named segments" are structural facts of this template, not swappable dataset variables. **The honest cost:** if a future client renames a plan or segment, this specific mapping needs a one-line edit. The fallback branch below exists so an unrecognized future value never silently disappears — it degrades to a generic truncation instead of breaking.

```dax
Segment Short Label =
SWITCH (
    DimCustomer[CustomerSegment],
    "SMB", "SMB",
    "Mid-Market", "Mid",
    "Enterprise", "Ent",
    LEFT ( SUBSTITUTE ( DimCustomer[CustomerSegment], "-", " " ), 4 )
)
```
On this dataset's actual values: "SMB" → "SMB", "Mid-Market" → "Mid", "Enterprise" → "Ent". Any segment name not explicitly listed falls to the last argument — a generic 4-letter truncation — rather than returning blank.

### 9.2 — Build the visual

1. Insert → **Column chart**. X=**856**, Y=**430**, Width=**198**, Height=**274**. *(Corrected from an earlier draft's x=848 — 848 is not a valid position on this report's 12-column grid; 856 is column 9's true start and aligns with the seam V1/V2 already share above it.)*
2. Axis: `DimCustomer[Segment Short Label]` — the new column, not the full `CustomerSegment` field. Values: the existing cumulative logo-churn-by-segment figures already validated in this project (SMB 35.4%, Mid-Market 26.0%, Enterprise 13.7%) — confirm the underlying measure name in your model and bind it here.
3. **Title: "Cumulative logo churn % by segment"** — not "Logo churn rate by segment" or any wording matching K3's KPI card. Subtitle: "Full 3-year period, all customers ever acquired", 10px `#94A3B8`.
4. Bars `#0F1B3D`. Data labels on, percentage format, 10px.
5. Sort descending by value.
6. **Tooltip: add `DimCustomer[CustomerSegment]` (the full name) as a field** so the shortened axis label never loses information — hovering any bar shows the real segment name in full.

**Checkpoint 9** — Axis shows "SMB," "Mid," "Ent." Values read SMB 35.4%, Mid-Market 26.0%, Enterprise 13.7%. Hovering any bar shows the full, unabbreviated segment name in the tooltip. Title unambiguously distinguishes this from the single-period `[Logo Churn Rate]` used in K3 — a viewer should not be able to confuse the two from the visible labels alone.

---

## Section 10 — V5b: Churned MRR by Plan

### 10.1 — New calculated column: hand-picked abbreviations, matching V5a's revised approach

Same problem as V5a, same resolution — a hand-picked mapping reads more naturally than pure truncation, accepted as the same named exception described in Section 9.1, with the same fallback protection:

```dax
Plan Short Label =
SWITCH (
    DimPlan[PlanName],
    "Starter", "Start",
    "Growth", "Grow",
    "Professional", "Prof",
    "Business", "Bus",
    "Enterprise", "Ent",
    LEFT ( DimPlan[PlanName], 4 )
)
```
On this dataset's actual plan names: Starter → Start, Growth → Grow, Professional → Prof, Business → Bus, Enterprise → Ent. Any plan name not explicitly listed falls to the last argument — a generic 4-letter truncation — rather than returning blank. Re-check this column's output whenever plan names change.

### 10.2 — Build the visual

1. Insert → **Column chart**. X=**1066**, Y=**430**, Width=**198**, Height=**274**.
2. Axis: `DimPlan[Plan Short Label]` — the new column, not the full `PlanName` field. Values: `[Churned MRR by Plan]` (existing USERELATIONSHIP-based measure — reuse unchanged, do not rebuild).
3. Bars `#0F1B3D`. Sort descending.
4. **Tooltip: add `DimPlan[PlanName]` (the full name) as a field** so the shortened axis label never loses information, same treatment as V5a.
5. Title "Churn by plan", subtitle "Churned MRR", 10px `#94A3B8`.

**Checkpoint 10** — Axis shows "Start," "Grow," "Prof," "Bus," "Ent." Hovering any bar shows the full plan name in the tooltip. Total bottom-row width: 16 to 1066+198=1264, flush with the report's standard content edge.

---

## Section 11 — Interactions (page-wide summary)

| Source | Target | Setting | Reason |
|---|---|---|---|
| V1 | Every other visual | **None**, both directions | Longitudinal question, structurally distinct — see 5.5 |
| V4 | V5a, V5b, V3, K1–K4 | **None** | No cross-filtering intended between churn breakdowns |
| V5a | V4, V5b, V3, K1–K4 | **None** | Same |
| V5b | V4, V5a, V3, K1–K4 | **None** | Same |
| V3 | Everything | **None** | Trend chart is read-only context |
| K1–K4 | Everything | **None** | Cards are not selectable, consistent with Pages 1–5 |
| Header Period, Segment | Every visual except V1's own internal logic (Period/Segment still apply to which customers are eligible, but do not alter V1's structural cross-filtering with other visuals) | **Filter** | Report-wide scope, synced |

---

## Section 12 — Drill-through

1. Page information → confirm "Allow drill through from" already includes the existing field: `DimDate[YearMonth]` (arriving from Page 1 and Page 2 month selections — already configured in this project's earlier build).
2. **No change needed to the drill-through field well itself** — this page's existing configuration is correct as-is.
3. Confirm Keep all filters: On.
4. **Explicitly verify V1 does not respond to this drill-through filter** — since V1 has no relationship to `DimDate` at all, arriving via month drill-through should narrow V3, V4, V5a, and V5b to that month, while V1 remains showing the full, unfiltered cohort history. This is intentional, not a defect — confirm it behaves this way rather than assuming.

**Checkpoint 12** — Drilling from a Page 1 or Page 2 month selection lands here with V3/V4/V5a/V5b narrowed to that month; V1 unchanged and full.

---

## Section 13 — Navigation, Bookmarks, Tab Order, Accessibility

1. Filters Open/Filters Closed bookmarks: confirmed rebuilt fresh in Section 3 (page-scoped, do not carry over from Page 5's copied shell).
2. Tab order: header → Period → Segment → Filters → K1 → K2 → K3 → V1 → V2 → V4 → V3 → V5a → V5b.
3. Alt text: V1 "Heatmap of customer retention percentage by signup quarter and cohort age in months." V2 "Comparison of health score and NPS between active customers and customers immediately before churn." V3 "Line chart of gross and net revenue retention over the full report period." V4 "Bar chart of churned MRR by recorded churn reason." V5a "Bar chart of cumulative logo churn percentage by customer segment." V5b "Bar chart of churned MRR by subscription plan."
4. Confirm no visual on this page relies on color as the only signal — re-check V2's ▼ glyph fix from Section 6.2 is actually applied, not just specified.

---

## Section 14 — Final QA

- **Gate 1 and Gate 2 both passed with documented evidence**, not assumed — re-confirm before sign-off.
- Palette check: only `#0F1B3D`, `#0F172A`, `#64748B`, `#3E5C8A`, `#94A3B8`, `#16A34A`, `#DC2626`, `#F4F5F7`, `#FFFFFF` anywhere on the page. `#DC2626` appears only on V2's before-churn values and K3's inverted-polarity delta — both genuine, justified uses, not decoration.
- No manual border or shadow on any content card — theme-inherited only. Header/nav flat.
- Bottom-row grid math re-verified: V4 (16, 408w) → V3 (436, 408w) → V5a (856, 198w) → V5b (1066, 198w) → ends at 1264, matching the report's standard content edge exactly.
- No visual on this page duplicates Page 3 (customer counts/trends), Page 4 (channel content, anywhere), or Page 5 (segment-value economics) in substance, even where a dimension (Segment, Plan) is legitimately shared.
- `ChurnReason` categories confirmed real, not invented (Section 8, step 1).
- Numeric reconciliation: K2 (GRR) and K1 (NRR) both reference the confirmed canonical measures from Gate 1 — re-open both formulas one final time and confirm nothing was altered during the build.
- Compare against Pages 1–5: header, nav, fonts, card treatment identical. This is confirmed as the final page — no seventh tab, no forward navigation beyond this point.

---

## Build-Complete Checklist

- [x] Gate 1 resolved — confirmed distinct (94.53% / 1.30%, sum 95.83%) — 4-card KPI strip built (NRR, GRR, Logo Churn Rate, Revenue Churn Rate) at 303px width each
- [ ] Gate 2 passed against corrected reference values (Age 0 = 100.00%, Age 3 = 88.78%, Age 6 = 81.00%, Age 24 = 62.86%, Age 30 = 61.05%) — original uncorrected values (87.09/82.95/73.45/69.37) formally retired, do not reintroduce
- [ ] `Signup Quarter` column built, 12 distinct values confirmed
- [ ] V1 heatmap complete, transparent (not zero) cells above the diagonal confirmed
- [ ] V2 health-before-churn measures reproduce corrected values (Health Score 54.71, NPS 12.91); active baseline confirmed at 68.88 / 36.89 — original uncorrected values (60.02/19.62) formally retired, do not reintroduce
- [ ] V2's ▼ glyph present — no color-only signal
- [ ] V3 spans the full 36-month period, no trailing-window mislabel
- [ ] V4's `ChurnReason` categories verified real before build
- [ ] V5a titled distinctly from K3's `[Logo Churn Rate]`
- [ ] V5b uses the existing USERELATIONSHIP measure, unmodified
- [ ] Bottom-row coordinates match the corrected grid math (856, not 848)
- [ ] All interactions set to None per Section 11 — no unintended cross-filtering
- [ ] Drill-through confirmed reaching V3/V4/V5a/V5b but not V1
- [ ] Tab order and alt text applied

## Remaining Phase 5 Items (post-Page-6, report-wide)

- `.pbit` template packaging with documentation
- Two never-assigned measures from earlier governance (`Net New MRR YTD`, `Rolling 12M Net New MRR`) — assign a home or formally retire
- Optional Page 1 trim: the Executive Insight card's "Net MRR Change" row, flagged as restating a KPI, never actioned
- Revenue-basis cohort retention (customer-basis only shipped on this page, by design — see original Page 6 architecture notes)
- Target/benchmark layer (schema designed, not implemented — ships empty for client population)
- Full report walkthrough: confirm every page's Period control, KPI delta wording, and filter flyout content are internally consistent now that all six pages exist side by side
