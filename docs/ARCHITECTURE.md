# Architecture

Why this is six pages rather than one, what each page owns, and which decisions are frozen.

---

## 1. The governing rule

> **Every metric has exactly one canonical measure and exactly one home page. A metric appearing on a second page is a documented exception, never drift.**

This matters because the failure mode of a multi-page report is not an incorrect number — it is the *same* number appearing three times with slightly different filter contexts, after which nobody trusts any of them. A written ownership matrix, checked before each page was designed and again when it was audited, is what prevents that.

The full matrix with per-metric reasoning lives in [`Metric_Page_Architecture_Control.html`](Metric_Page_Architecture_Control.html) — a self-contained HTML document; download it and open it in a browser.

> ⚠️ **That document predates Phase 5.** It still cites the health-at-churn values 60.02 / 19.62, which were tested, failed to reproduce, and were retired in favour of 54.71 / 12.91. Read it for the *ownership model*, not for figures. [VALIDATION.md §3.2](VALIDATION.md#32-health-at-churn) has the correction.

---

## 2. Page ownership

| Page | Question | Owns | Explicitly does not carry |
| --- | --- | --- | --- |
| **1 · Executive Overview** | How is the business performing overall? | MRR, ARR, MRR Growth %, Active Customers (Tier 1); the Executive Insight narrative card (exclusive) | Any dimensional breakdown, any matrix, anything requiring interaction to interpret |
| **2 · Revenue & Growth** | What is driving MRR movement? | All MRR movement — New / Expansion / Contraction / Churned, Gross New MRR, Net MRR Change, growth composition | Customer counts as a subject; unit economics |
| **3 · Customer Growth** | Is the base growing, and how does acquisition compare to loss? | Customer counts, Customer Growth %, Logo Churn Rate, acquisition-vs-churn comparison | Revenue retention, cohort analysis, churn *reasons* |
| **4 · Unit Economics** | Is growth economically efficient? | CAC, LTV, LTV:CAC, CAC Payback, cost per lead, conversion — all by channel | Any non-channel dimension; customer-level analysis |
| **5 · Segment Analysis** | Which customer groups drive the business? | Group economics — ARPA, MRR, Estimated LTV, plan mix by segment / industry / country | Churn rates, retention, cohorts (all Page 6); acquisition channel (Page 4) |
| **6 · Retention & Churn** | Are customers and revenue retained, and who leaves? | Cohort retention, GRR, NRR, Revenue Churn Rate, churn reasons, churn by segment and plan, health at churn | Customer growth as a subject (Page 3); acquisition channel entirely |

### Deliberate repeats, and why

| Metric | Home | Also appears | Justification |
| --- | --- | --- | --- |
| `MRR` | P2 | P1 (headline), P5 (by group) | P1 shows the company total as context; P5 decomposes it dimensionally. Different questions, one measure |
| `Logo Churn Rate` | P3 | P1, P6 | P3 owns it as a trend; P1 shows the current value; P6 cuts it by segment for churn diagnosis |
| `NRR` | P6 | P1 | P1 shows the number; P6 shows it against GRR, which is the comparison that makes it mean anything |
| `ARPA` | P1 | P4, P5, P6 | A denominator in LTV (P4) and a group economic (P5) before it is a headline |
| `GRR` | **P6** | P1 (as text inside the Insight card only) | P1 never gives it a card — mentioning a figure in a sentence is not owning it |

---

## 3. The model

```
DimDate ──────────┬──→ FactSubscription ──────→ DimPlan
                  ├──→ FactSubscriptionEvent ╌╌→ DimPlan   (inactive ×2)
                  ├──→ FactMarketing ────────→ DimAcquisitionChannel
                  └──→ FactCustomerHealth              ↑
                                                       │
DimCustomer ──────┬──→ FactSubscription                │
                  ├──→ FactSubscriptionEvent           │
                  ├──→ FactCustomerHealth              │
                  └───────────────────────────────────-┘
```

13 relationships, 10 active, all single-direction from fact to dimension.

### The three inactive relationships

| Relationship | Role | Activated by |
| --- | --- | --- |
| `FactSubscriptionEvent → DimPlan` | `NewPlanID` | `USERELATIONSHIP`, per measure |
| `FactSubscriptionEvent → DimPlan` | `PreviousPlanID` | `USERELATIONSHIP` — e.g. `Churned MRR by Plan` |
| `FactSubscription → DimDate` | `EndDate` | `USERELATIONSHIP`, where end-date context is needed |

**An event carries two plan keys.** Making either one active would silently make the other invisible, so neither is. Any new plan-level measure written against event data must state which key it means. This is the single most important thing to know before extending the model.

### Why `FactMarketing` doesn't reach customers

It is at **channel × month** grain, not customer grain. There is no `CustomerID` to join on, and inventing one would be fabrication. It reaches the rest of the model through `DimAcquisitionChannel` and `DimDate` only.

**Consequence, visible in the report:** Page 4's Segment slicer is present for visual consistency across pages but does **not** filter that page's channel-level marketing metrics — `FactMarketing` has no segment dimension to filter on. This is a documented, accepted deviation rather than a bug.

---

## 4. Reusability mechanisms

Each of these is the specific thing that makes the file a template rather than a report:

| Mechanism | Implementation | Verified |
| --- | --- | --- |
| **Parameterised source** | `SourceFilePath` Power Query parameter; all 7 tables use `Excel.Workbook(File.Contents(SourceFilePath), null, true)` | Repointed at a different file and refreshed end-to-end |
| **Relative date anchoring** | Every period measure resolves against `MAX(DimDate[Date])` | DAX inspection across all period measures — no `TODAY()`, no hardcoded dates |
| **Dynamic category handling** | `RANKX`-based Top-N on live measures, not fixed category lists | Cutoff changed live; dependent visuals recalculated |
| **Adjustable assumptions** | Three `GENERATESERIES` what-if parameters, read via `SELECTEDVALUE` with defaults | Sliders moved; LTV / LTV:CAC / Payback responded, CAC correctly did not |
| **Graceful degradation** | Short-label fallbacks for unseen category names | Paper-traced against two hypothetical names; both degrade sensibly |

### The named exception

`DimCustomer[Segment Short Label]` and `DimPlan[Plan Short Label]` are hand-picked abbreviations (`Mid-Market` → `Mid`, `Enterprise` → `Ent`) used where Page 6's narrow columns cannot fit full names. This is a **deliberate, named exception** to the project's no-hardcoded-category-text rule, justified because the category structure is treated as intrinsic to this template. A `SWITCH` fallback handles unrecognised names, though collision-free uniqueness is not structurally guaranteed the way the two-word-initials case is.

---

## 5. Design system

Frozen across all six pages without exception.

| | |
| --- | --- |
| Canvas | 1280 × 720, 12-column grid, 16 px margins, 93 px columns, 12 px gutters |
| Spacing scale | 4 / 8 / 12 / 16 |
| Zones | Header 16–56 · navigation 60–90 · Tier 1 content from ~102 |
| Type | Segoe UI. Titles 14 Semibold · subtitles 10 · KPI values 22–30 Semibold · content floor 10 px |
| Navy | `#0F1B3D` — all data marks by default |
| Slate / Neutral | `#64748B` · `#94A3B8` — secondary series, labels |
| Background / Cards | `#F4F5F7` · `#FFFFFF` |
| Positive / Negative | `#16A34A` · `#DC2626` — **reserved for genuine deltas only** |
| Accent | `#F59E0B` — callouts only |

Machine-readable: [`../powerbi/SaaS_Dashboard_Theme.json`](../powerbi/SaaS_Dashboard_Theme.json).

**Colour carries meaning, never decoration.** Series are navy by default; green and red appear only where a number genuinely moved in a good or bad direction. The cohort heatmap uses **single-hue navy intensity only** — never a red-to-green diverging ramp — because retention magnitude is not a judgement, and a diverging scale would both imply one and fail for colour-vision-deficient readers.

### The KPI card pattern

Every KPI card is **three stacked visuals** — value, delta, caption — rather than one Card visual with a reference label. Card (new)'s reference-label conditional formatting proved unreliable during the build, so the pattern trades object count for correctness. This is the main reason the model carries 180 measures for ~53 business metrics, and a contributor to the page-load times in [VALIDATION.md §8](VALIDATION.md#8-performance). It was re-examined during performance work and deliberately kept.

---

## 6. Page structure

14 pages in the file: **6 dashboard pages** plus **8 tooltip pages** (report-page tooltips bound to specific visuals).

| # | Page |
| --- | --- |
| 1–6 | Executive Overview · Revenue & Growth · Customer Growth · Unit Economics · Segment Analysis · Retention & Churn |
| 7–14 | Tooltip pages: MRR Trend · Movement · Active Customers · MRR by Plan · MRR Movement · Unit Economics · Segment Analysis · Revenue and Churn |

Navigation between the six is plain **Page navigation** actions — confirmed, not bookmark-driven. Bookmarks are reserved for the filter flyout.

---

## 7. Build record

Each page was built from a click-by-click SOP, then audited against the actual build — not against the SOP. **That audit step is where most DAX bugs in this project were found**, including context-sensitivity bugs invisible to single-month testing but that surfaced under real page-level filter widths.

Per-page SOPs: [`page-sops/`](page-sops/). They describe the final working build, revised where implementation diverged from the original plan.
