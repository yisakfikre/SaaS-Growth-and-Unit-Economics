# Validation report

**Artifact:** `powerbi/SaaS_Growth_Unit_Economics.pbit` · built 2026-09-30 · 14 pages (6 dashboard + 8 tooltip) · 180 measures
**Dataset:** `data/SaaS_Growth_Unit_Economics_Dataset.xlsx` · synthetic, seeded (42 / 43) · 2023-01-01 → 2025-12-31

This document states what was checked, how, what it found, and — in §7 — what it does not cover. The underlying step-by-step records are in [`qa-records/`](qa-records/), unedited.

---

## 1. What kind of validation this is

**Documented human verification against live report values and independent hand-derivation.** Each measure was inspected, reasoned about, and where possible cross-checked against a separately-derived figure or a reconciliation between two visuals that should agree.

**It is not an automated harness.** There is no script in this repository that recomputes the figures and asserts them. The Python used to generate the synthetic dataset is not published here, so **the numbers cannot be independently recomputed from this repository alone.** That is a genuine limitation of this project's evidence, stated plainly rather than implied away.

What makes the record credible is not the pass count — it is that two documented "validated" reference values were tested, **failed**, and were retired rather than kept.

---

## 2. Status vocabulary

The measure inventory uses three statuses, and the distinction is load-bearing:

| Status | Meaning |
| --- | --- |
| **Validated** | Confirmed live — a reported value matched an expected one |
| **Needs Re-test** | Worked or was specified, but a later change means it must be re-checked |
| **Unverified** | Built per SOP, never confirmed live. **Genuinely unknown, not assumed passing** |

Full inventory: [`qa-records/Phase5_Step1_Measure_Validation.md`](qa-records/Phase5_Step1_Measure_Validation.md).

---

## 3. The two corrections — the most useful thing this process produced

### 3.1 Cohort retention

Four reference values carried through project documentation as "validated" were tested against independently-built DAX. **They did not reproduce.**

| Cohort age | Original documented | Tested and adopted |
| --- | --- | --- |
| 0 months | — | 100.00% |
| 3 months | 87.09% | **88.78%** |
| 6 months | 82.95% | **81.00%** |
| 24 months | 73.45% | **62.86%** |
| 30 months | 69.37% | **61.05%** |

The 24-month figure was off by more than ten percentage points. The replacements were cross-confirmed by two independently-reasoned methods. **The originals were retired, not re-derived to fit** — the project record explicitly forbids reintroducing them.

### 3.2 Health at churn

The same pattern, investigated further. Two candidate definitions of "health before churn" were tested against the documented reference of 60.02 / 19.62:

| Definition | Health Score |
| --- | --- |
| Last snapshot before churn | **54.71** |
| Average across full tenure to churn | 64.25 |
| *Documented reference* | *60.02* |

**The reference sat between the two tested definitions rather than matching either** — which rules both out as its original method rather than validating one. No provenance for the original figure was found.

**Last-snapshot-before-churn was adopted as canonical** on the grounds that it answers the question the visual actually asks. Adopted values: **Health Score 54.71 · NPS 12.91**, against an active-customer baseline of **68.88 · 36.89**. The page is labelled *"before churn"*, not *"at churn"*, because under this definition those are different claims.

### 3.3 GRR and Revenue Churn Rate are not inverses

An earlier assumption that `[GRR]` and `[Revenue Churn Rate]` summed to 100% — which would have made one of them redundant — was tested against live values and **disproven**. GRR carries contraction; revenue churn does not. See [METRIC_DEFINITIONS.md §5.1](METRIC_DEFINITIONS.md#51-grr-and-revenue-churn-rate-are-not-arithmetic-inverses). This is why Page 6's KPI strip carries four cards.

---

## 4. Verified figures

Confirmed live during validation:

| Measure | Value | How |
| --- | --- | --- |
| `GRR` | 94.5% | Matches independent hand-derivation at 94.53% |
| `Cohort Retention %` | 88.78 / 81.00 / 62.86 / 61.05 | Cross-confirmed by two independent methods |
| `_Health Score Before Churn` | 54.71 | Definition tested against two alternatives |
| `_NPS Before Churn` | 12.91 | As above |
| `New Customers` | — | Confirmed across three different filter windows |
| `New Customer Rate` | — | Reconciled to `Customer Growth %` via the July 2024 churn-wave residual |
| Period anchoring | all measures | DAX inspection across Steps 1 and 4: every period measure uses `MAX(DimDate[Date])`; **no `TODAY()`, no hardcoded dates** |

---

## 5. Structural findings that changed the design

Validation that only confirms the plan is not validation. These are the cases where testing forced a change.

### 5.1 The `.pbit` would not have worked

**Found at Step 7, after every other reusability claim had been checked.** A `.pbit`'s "prompt on open" behaviour belongs to **Power Query parameters** — a different mechanism from the What-if parameters this project had built throughout. Every table's Source step still pointed at a hardcoded path.

> Exporting to `.pbit` before this fix would have produced a template with nothing for a new user to plug their own data into — the single most important promise of "reusable template", undeliverable, despite every other reusability decision being sound.

**Fixed:** `SourceFilePath` built as a Power Query parameter; all seven tables repointed to `Excel.Workbook(File.Contents(SourceFilePath), null, true)`; the parameter changed to a different file, refreshed end-to-end, and confirmed. This is the only claim in the project that *required* breaking the file to prove.

**And that test caused an incident worth recording:** the revert pointed at a similar-but-not-identical backup file. Page 1's waterfall changed shape and Page 4 went nearly blank. Several hours were spent ruling out relationships, slicer state and data types before the actual cause — a path that differed by a folder — was found by character-by-character comparison. The model had been loading a *different file* perfectly correctly the whole time.

### 5.2 Two hard Power BI limits, discovered by testing

| Finding | Consequence |
| --- | --- |
| **Native Top-N never produces an "Other" bucket**, for any visual type | A true "Other" requires a custom rank-and-relabel mechanism |
| **No measure-based filter can be applied at page or report level** — only at visual level | Page 4's Top-N cutoff is applied as an identical visual-level filter on three visuals, synchronised **by hand** |

A calculated-column alternative for the second was tried and rejected: it would freeze the ranking at the last model refresh and be silently wrong the moment the Period filter moved.

### 5.3 A real duplicate-measure bug

`Blended Marketing CAC` and `Blended Marketing CAC (P4)` both existed — the exact class of duplication the project's metric-ownership rule is designed to prevent, found in the page where it mattered most.

### 5.4 A measure deleted in error, and recovered

`_Active Customers Net Change` was grouped as a delete candidate in an earlier pass of the inventory. It was not insight-card-only — it is the direct source of the "+93 net vs prior month" caption on **two** pages. Deleting it broke both captions; it was reconstructed and re-validated. Recorded because the inventory itself was wrong, and the record should say so.

---

## 6. Open and unresolved

Listed because a validation report that lists only closures is not a validation report.

| Item | State |
| --- | --- |
| **Pages 1–2 "Top N + Other"** | Both pages were specified as "Top N + Other" **before** §5.2's finding. Page 4 was re-checked and corrected. **Pages 1–2 were not re-checked.** It is entirely possible one or both carry the same silent gap |
| **Page 5 share re-summing** | `_MRR Share of Total` was validated against Page 2's two-option field parameter. Page 5's three-option parameter is **untested** — the open check is whether the five plan shares total 100% *within a selected group*, not against the company-wide baseline |
| **Documentation vs. build: the "Other" bucket** | The Step 6 record states that exclude-only behaviour (no "Other" row) was accepted as Page 4's final design. **The shipped model contradicts this** — a `DimAcquisitionChannel[Channel Group]` calculated column produces an "Other" bucket via `RANKX ≤ 6`, and the published Page 4 image shows an "Other" row and bubble. The build moved on; the Step 6 record was not updated. **The build is correct; the record is stale.** Flagged rather than silently reconciled |
| **`Channel Group` is a calculated column** | It therefore ranks channels **as of the last model refresh**, not as of the current Period selection — the exact freezing behaviour that was rejected for the visual-level filter. Correct for a stable "Other" label; worth knowing before relying on it under a narrowed period |
| **`Cohort Revenue Retention %`** | Exists in the model, placed on no page, never validated. Unfinished work left in the file |
| **Some measures never live-confirmed** | The inventory marks these **Unverified** rather than assuming them correct |

---

## 7. What this validation does not cover

Stated explicitly, because the boundary matters more than the passes.

- **It validates arithmetic and filter behaviour, not business truth.** The figures are correctly computed from the data. The data is synthetic and describes nothing real.
- **It cannot be re-run.** No published script recomputes these figures. A reader can inspect the DAX, open the file and check the numbers by hand — which is what was done — but cannot execute a verification suite.
- **The LTV model is not validated as realistic, only as correctly implemented.** `LTV = ARPA × margin ÷ smoothed churn` returns a blended LTV:CAC of 35.7× on this data. That is the formula working as specified on a low-churn simulation, **not** a defensible SaaS benchmark. See [METRIC_DEFINITIONS.md §6.3](METRIC_DEFINITIONS.md#63-ltv-formula-limits).
- **Causation is out of scope entirely.** `ChurnReason` records a category someone selected. Health scores drive churn *by construction* in the simulation. Neither establishes why a real customer would leave.
- **Report-layer behaviour was verified by inspection**, not instrumentation — tooltips, cross-filter settings, drill-through targets and conditional-format bindings were checked by hand in Power BI Desktop.
- **Replacement-data testing was partial.** Shape changes were tested against the current file (narrowed periods, sparse groups, parameter movement). A genuinely different dataset — a fourth segment, a sixth plan — was **not** tested; it was judged disproportionate to simulate pre-emptively. See [`qa-records/Phase5_Step6_Data_Contract_Testing.md`](qa-records/Phase5_Step6_Data_Contract_Testing.md).

---

## 8. Performance

Cold navigation to heavier pages: **9.5–14.5 seconds**.

Investigated rather than accepted on sight. Bookmark-driven navigation was ruled out by checking the Action Type (plain Page navigation). A blank-page test showed navigation itself is near-instant regardless of trigger — **the delay scales with the destination page's visual-object count**. Performance Analyzer logs *zero events of any kind* during the gap, placing the bottleneck below its instrumentation. Memory pressure (Power BI Desktop at 6.9 GB of 16 GB, ~84% system-wide) was identified as a plausible contributor but **not conclusively proven**.

The heaviest single visual is Page 6's cohort heatmap at **2,457 ms render** against 58 ms of DAX — per-cell conditional formatting across ~78 populated cells, accepted deliberately as the cost of the report's most information-dense visual.

Accepted for a 14-page file. Full detail: [`qa-records/Phase5_Step4_Performance.md`](qa-records/Phase5_Step4_Performance.md).

---

## 9. The seven-step record

| Step | Scope | Record |
| --- | --- | --- |
| 1 | DAX / measure validation — full inventory with per-measure status | [Step 1](qa-records/Phase5_Step1_Measure_Validation.md) |
| 2 | Functional and analytical QA — cross-page figure reconciliation | [Step 2](qa-records/Phase5_Step2_Functional_QA.md) |
| 3 | Filters, interactions, drill-through, bookmarks, navigation | [Step 3](qa-records/Phase5_Step3_Filters_Interactions_Navigation.md) |
| 4 | Performance | [Step 4](qa-records/Phase5_Step4_Performance.md) |
| 5 | Accessibility and UX | [Step 5](qa-records/Phase5_Step5_Accessibility_UX.md) |
| 6 | Data contract and replacement-data testing | [Step 6](qa-records/Phase5_Step6_Data_Contract_Testing.md) |
| 7 | PBIT packaging and template readiness | [Step 7](qa-records/Phase5_Step7_PBIT_Template_Readiness.md) |

These are working records, not polished summaries. They contain the dead ends.
