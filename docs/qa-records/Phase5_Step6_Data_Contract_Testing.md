# Phase 5 — Step 6: Data Contract / Refresh / Replacement-Data Testing

SaaS Growth & Unit Economics · Scope note: full dataset replacement judged disproportionate to what's actually at risk — this step tests whether the template's "not hardcoded" claims hold under changed data *shape*, using the current file wherever that's sufficient, and flags the few items that genuinely need new data as deferred rather than blocking.

---

## Already Closed — by direct inspection, no live test needed

| Claim | How it was confirmed |
|---|---|
| `Segment Short Label` / `Plan Short Label` fallback branches handle names never seen during design | Paper-traced against two hypothetical unseen names ("Growth-Stage" → "Grow", "Premium" → "Prem") — both degrade sensibly, no blank or broken output |
| Period anchoring uses the data's own last date, never a hardcoded date or `TODAY()` | Directly confirmed via DAX inspection multiple times across Steps 1 and 4 — every measure checked uses `MAX(DimDate[Date])` |

## Part A — Live tests on the current file, no data changes

### A1 — Top N recalculates, channel/segment names aren't hardcoded

**Status: closed, but this test surfaced two real, previously-undiscovered Power BI limitations that changed the design — worth recording in full, not just the final confirmation.**

**Finding 1 — native Top N never produces an "Other" bucket, for any visual type, and never has.** Confirmed against multiple sources. The original design across Pages 1, 2, and 4 assumed "Top N + Other" was a single native capability; it isn't — Top N only excludes everything beyond N, silently. Decision made: accept exclude-only behavior (no "Other" bubble/row) as the final design on Page 4, rather than build the heavier custom rank-and-relabel mechanism a true Other bucket would require.

**Finding 2 — no filter that depends on a measure can ever be placed at the page or report level, only at the visual level; this is a hard structural rule, not a version limitation.** Attempted a page-level `_Channel Rank` measure filter specifically to guarantee the scatter, table, and Where Spend Sits panel could never drift out of sync with each other. Not possible — page-level filters only accept column values, and a genuinely dynamic (Period-slicer-responsive) ranking can only exist as a measure. A calculated-column alternative was also tried and rejected: it would freeze the ranking at whatever it was during the last model refresh, silently wrong the moment the Period filter changed.

**Final resolution:** `_Channel Rank` (`RANKX` on `Marketing Spend (P4)`) built as a measure, applied as an identical **visual-level** Basic filter (≤ 6) on all three visuals individually — the only mechanism Power BI actually supports for this combination. **Documented, standing maintenance note:** if the cutoff ever changes from 6, it must be changed in all three places by hand; nothing enforces this automatically. Confirmed live: scatter, table, and panel now show the same six channels.

### A2 — Parameters are genuinely adjustable, not disguised constants
1. Page 4: move the **Assumed Gross Margin %** slider to a different value. Confirm LTV, LTV:CAC, and Payback all move; confirm CAC does not (it has no margin dependency).
2. Move **LTV:CAC Target** and **Min Volume** sliders. Confirm the scatter's reference line and marker colors respond, and the Where Spend Sits panel's grouping updates.
3. Return all three to their defaults when done.

### A3 — Sparse-group handling doesn't break on a small category
1. Page 5 → open the Country filter in the flyout.
2. Select a genuinely low-volume country from the real data (any one with a small customer count).
3. Confirm V1 renders a single small bubble without error, V3 shows exactly one row with a correct (non-blank, non-error) total, and no visual shows a divide-by-zero or blank state.
4. Clear the filter when done.

### A4 — Narrow date range degrades gracefully (cheap proxy for "less history")
This can't literally shrink the dataset, but narrowing the Period filter is a fair, cheap proxy for "what if this template only had 12 months of history instead of 36" — the case a new client would actually start in.
1. On Page 6, narrow the Period slider to roughly the most recent 12 months.
2. Confirm the cohort heatmap still renders sensibly — expect it to show fewer populated cohort-quarters (younger data has fewer completed cohorts), not an error or a blank grid.
3. Confirm the GRR vs NRR trend line still draws correctly over the shorter window.
4. Check Page 1–3's trend charts under the same narrowed range — confirm none of them assume a minimum number of months to render.
5. Reset the Period filter to full range when done.

## Part B — Genuinely requires new data, documented and deferred

| Claim | Why it can't be tested on current data | Status |
|---|---|---|
| Behavior with a 4th customer segment | Current data has exactly 3 segments, structurally — no filter can simulate a 4th | **Deferred** — would require adding a new `DimCustomer` segment value with real supporting rows, a genuine data change, not a test |
| Behavior with 6+ plans | Current data has exactly 5 plans, structurally | **Deferred** — same reasoning, would need a new `DimPlan` row with corresponding subscriptions |

**Recommendation, unchanged from the proposal:** these two are lower priority given cost-to-value. If this template is ever actually handed to a second client with a different segment or plan count, that engagement is the natural point to confirm these for real — building synthetic data now, purely to pre-prove a scenario that may never occur exactly this way, isn't a good use of effort at this stage. Documented here so the gap is known, not silently absent.

## Status: Frozen

A1–A4 all confirmed live on the file. Part B's two items (4th segment, 6th plan) remain explicitly deferred, documented above, not blocking freeze.

**Standing item carried forward from A1, not closed by this step but worth flagging for the documentation package in Step 8:** Pages 1 and 2 use the identical "Top N + Other" language in their own design specs, built before this limitation was discovered here. Neither was re-verified against this finding — both should get the same check (does "Other" actually appear, or silently not) before this project is considered fully audited.
