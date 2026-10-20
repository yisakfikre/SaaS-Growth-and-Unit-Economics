# Phase 5 — Step 5: Accessibility and UX QA

SaaS Growth & Unit Economics · **FROZEN**

---

## What This Step Covered

Four commitments already written into every page's SOP, verified rather than assumed built:

1. **Tab order** — confirmed per page, matching each SOP's specified sequence.
2. **Color never the only signal** — confirmed; Page 6's Health-at-Churn ▼ glyph (added during Page 6's own build) is the one place this had already been tested for real.
3. **10px content-text floor** — confirmed; Pages 5 and 6's matrix headers were the two places this was actually corrected during build.
4. **Alt text** — found to be the one item never actually specified with real wording anywhere except Page 6. Rebuilt from scratch across all six pages — see below.

## Alt Text — Final State

**Scope rule applied:** alt text on every object conveying meaningful information — charts, tables, matrices, KPI values, interactive controls. Not applied to decorative shapes, dividers, or container rectangles, per Microsoft's own accessibility guidance ("every object that conveys meaningful information," not literally everything).

**Dynamic vs. static rule applied:** any alt text stating a number, or making a ranking/comparison claim that could flip under a filter, is bound to a DAX measure via the Alt text field's `fx` option — never a hardcoded string that can go stale. Anything describing a structural or mathematically-guaranteed relationship (e.g., "NRR above GRR" — true by definition, expansion only adds) is safe as static text.

**19 dynamic Alt Text measures built**, grouped under a dedicated `Alt Text` display folder in the Fields pane, separate from the report's business-logic measures:

*Shared across pages (one measure, bound on every card showing that KPI):*
`_MRR Alt Text` · `_ARR Alt Text` · `_Active Customers Alt Text` · `_Customer Growth Alt Text` · `_Logo Churn Alt Text` · `_NRR Alt Text`

*Page-specific KPIs:*
`_MRR Growth Alt Text`, `_ARPA Alt Text` (P1) · `_Net MRR Change Alt Text`, `_Gross New MRR Alt Text` (P2) · `_New Customers Alt Text`, `_Churned Customers Alt Text` (P3) · `_Blended CAC Alt Text`, `_LTV Alt Text`, `_LTVCAC Alt Text`, `_Payback Alt Text` (P4) · `_GRR Alt Text`, `_Revenue Churn Rate Alt Text` (P6)

*Ranking-claim charts (upgraded from static after review — a filter could otherwise make these false):*
`_MRR by Plan Alt Text` (P1) · `_Churn by Reason Alt Text`, `_Churn by Plan Alt Text` (P6)

**Full page-by-page placement** (every visual, every control, static text or measure name) — see the consolidated table delivered in-chat during this step; not reproduced here to avoid duplicating ~50 rows, but every item in it was confirmed applied.

**Two corrections made during final review, worth remembering if this pattern repeats on a future build:**
1. A "Group By selector" alt-text item was mistakenly listed for both Page 5 and Page 6 — Page 6 has no such parameter; its dimensions are fixed, separate visuals. Removed from Page 6.
2. "Cumulative logo churn % by segment" (Page 5) makes a ranking claim (smaller segments churn more) that was never formally vetted for filter-safety the way the other three ranking charts were. Left as static text — the practical risk is low, since narrowing the Segment filter just leaves one bar rather than producing a false comparison — but this is flagged honestly as a lighter check than the other three received, not a confirmed-safe claim.

## Standing Lesson From This Step

The dynamic-vs-static gap was caught the same way the cohort retention and health-at-churn reference values were caught earlier in this project: a static claim survived unquestioned until a real, live discrepancy (Page 4's 12-month-window figures visibly disagreeing with hardcoded alt text) forced a proper check. The same principle applies here as everywhere else in this build — a plausible-looking static value is not a substitute for one that's provably tied to the live model.

## Status: Frozen
