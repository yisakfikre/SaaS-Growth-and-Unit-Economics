# Phase 5 — Step 3: Filters, Interactions, Drill-through, Bookmarks, Navigation

SaaS Growth & Unit Economics · **FROZEN** · Verified during each page's own build/checkpoint process rather than a separate formal pass — this document consolidates those already-confirmed rules into one reference rather than re-testing them.

---

## 1. Header Filters, Per Page

| Page | Period control | Segment slicer | Flyout contents |
|---|---|---|---|
| P1 | `DimDate` range slider, anchored to Last date | Standard, Filter to all | Country, Industry, Channel, HealthScore (range), Plan |
| P2 | Same as P1 | Standard | Reused from P1 |
| P3 | Same as P1 | Standard | Country, Industry, Channel, HealthScore |
| P4 | **Page-4-specific `Period Options` disconnected table** ("Last 12 complete months") — deliberately different mechanism, not the `DimDate` slider | Present but **inert by design** — Edit Interactions = None to every P4 visual, since `FactMarketing` has no segment column | Target (LTV:CAC), Min Volume — parameters, not standard dimension filters |
| P5 | `DimDate` range slider — standard, **not** P4's mechanism | Standard, Filter to all | Industry, Country only — Plan/Channel/Health explicitly excluded, sync cleared |
| P6 | `DimDate` range slider — standard | Standard | Industry, Country, Churn Reason — Plan/Channel excluded |

**Rule confirmed holding throughout:** every flyout is rebuilt fresh per page (Filters Open/Filters Closed bookmarks are page-scoped and never carry over, even when the shell group itself is copied from the prior page).

## 2. Drill-through

| Target page | Field | Sources | Notes |
|---|---|---|---|
| P5 | `DimCustomer[CustomerSegment]` | Page 2's matrix, when its Plan/Segment parameter = Segment | `DimPlan[PlanName]` was **removed** as a trigger from both Page 1's bar chart and Page 2's matrix — `DimPlan → FactSubscriptionEvent` is inactive, so a Plan-based drill-through would have silently failed to filter P5's measures. Verified live in Step 1: Filters pane shows the exact segment on arrival, Group By switching post-arrival correctly narrows within that segment. |
| P6 | `DimDate[YearMonth]` | Page 1's Active Customers trend, Page 2's composition chart | Confirmed **not** reaching V1 (cohort heatmap) — intentional, that visual has no relationship to `DimDate` at all. |

## 3. Bookmarks

- **Filters Open / Filters Closed** — present on every page with a flyout (all six). Data unchecked, Display + Current page checked. Rebuilt fresh per page.
- **P4 Reset Defaults** — Page 4 only. Resets the Target and Min Volume parameter sliders to their defaults (3.0, 30). Data checked, scoped via Selected Visuals to just those two objects — deliberately not using the native Clear-slicer action, since a single-value numeric parameter slider has no reliably-defined "cleared" state.

## 4. Interaction Rules Established Across the Report

**Headline/KPI cards never narrow** — every Tier 1 / KPI card on every page is set to None toward every other visual on its page. A summary figure that could be accidentally filtered into a misleading sub-view defeats its own purpose.

**The `DimPlan → FactSubscriptionEvent` gap governs several rules, not just one:**
- P1: MRR by Plan chart → rest of page = None (documented limitation, not a bug)
- P2: Matrix → 3 charts = Filter for Segment, **not** for Plan
- P5: Plan matrix (V2) → everything else = None, mandatory — Plan cannot validly filter customer-count measures
- P6: same reasoning applies to why V5b (Churned MRR by Plan) never cross-filters V4 or V5a

**Deliberately inert-by-design controls, not oversights:**
- P4's Segment slicer — present for visual consistency across the report, set to None everywhere on that page since `FactMarketing` has no segment column
- P6's V1 (cohort heatmap) — None in both directions toward every other visual on the page, since it answers a structurally different, longitudinal question than everything else there

**Standard "drill down, then look up" pattern**, used consistently: a detail-level click (a table row, a bubble) Filters the more summary-level companion visual and Highlights (not Filters) the peer visual it came from — e.g., P5's V3 row → V2 = Filter, → V1 = Highlight.

## 5. Navigation

`Header_Nav_Global` group, copied page to page unmodified except the selected-tab underline and its color. Six tabs in fixed order: Executive Overview → Revenue & Growth → Customer Growth → Unit Economics → Segment Analysis → Retention & Churn. Header/nav band renders flat (no shadow) per the frozen design-system amendment; content cards beneath it keep the theme-supplied shadow.

---

## Recommended, Not Blocking

Everything above was verified within each page's own build session. The one thing not explicitly exercised: a single continuous click-through, Page 1 → 6 and back, in one sitting — the class of issue (a bookmark state leaking across a page switch, a sync-slicer behaving differently after several page changes) that sometimes only surfaces that way. Worth doing once before final delivery as a cheap confidence check, not a re-verification of this step.

**Status: Frozen.**
