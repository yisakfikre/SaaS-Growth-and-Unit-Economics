# Phase 5 — Step 4: Performance

SaaS Growth & Unit Economics · Based on live Performance Analyzer exports, all six pages, single refresh capture per page (files mapped by visual composition: File 1→P1, 2→P2, 3→P3, 4→P4, 5→P5, 6→P6)

---

## Headline Finding

**No DAX or data-model performance problem exists anywhere in this report.** Every analytical visual's actual DAX execution time, isolated from rendering and queue wait, is under 160ms — most in the 10–60ms range. This includes both measures flagged as suspects going into this step:

| Suspect | DAX execution time | Verdict |
|---|---|---|
| Cohort Retention % (Page 6 V1) — row-by-row `FILTER` over `DimCustomer` | 58ms | Cleared |
| `USERELATIONSHIP`-based Plan measures (P5 V2, P6 V5b) | 9–34ms | Cleared |

**No DAX changes are warranted anywhere in this model as a result of this step.**

## What's Actually Costing Time: Rendering, Concentrated in Two Places

| Visual | DAX | Render | Verdict |
|---|---|---|---|
| **Page 6 — Customer retention by cohort (V1)** | 58ms | **2,457ms** | Largest single cost in the report. Not a data or formula issue — almost certainly the per-cell gradient conditional formatting across the 12×12 grid (~78 individually-styled cells). |
| Page 4 — Channel Detail (table) | 77ms | 751ms | |
| Page 4 — Table below target | 76ms | 719ms | Three P4 tables rendering sequentially adds up cumulatively, though none is alarming alone. |
| Page 4 — Table above target | 74ms | 714ms | |

### Investigated and resolved
Checked live: **Background color conditional formatting is on; Font color conditional formatting is off** — meaning cell numbers currently render visibly in their default color, on top of the background gradient, contributing at least some real portion of the 2,457ms render cost. (Correction along the way: the original SOP cited a "Show cell value text" toggle that doesn't exist in Power BI — no such setting is real; the genuine technique to hide matrix cell text is a second Font color conditional-formatting rule matching the background's colors, which was never applied here. The SOP has been corrected to describe the real mechanism.)

**Decision: keep visible text (Option A), confirmed deliberately.** The background color communicates direction (higher/lower) at a glance; the visible numbers add precision on top rather than duplicating that signal — both layers earn their place. The 2,457ms render cost is accepted as the reasonable cost of the report's richest, most information-dense visual, not treated as a defect to engineer away. No further action on this item.

## A Separate, Informational Pattern — Not a Performance Problem

Every page's Card visuals showed uniformly elevated total durations, closely tracking each page's total Card *count* rather than any individual card's own query cost:

| Page | Total visual objects | Of which Card | Max total duration |
|---|---|---|---|
| P1 | 63 | 33 | 1,514ms |
| P2 | 40 | 14 | 1,458ms |
| P3 | 40 | 17 | 1,218ms |
| P4 | 57 | 17 | 1,949ms |
| **P5** | **25** | **1** | **1,242ms** |
| P6 | 57 | 17 | 3,524ms |

Page 5 — the only page with no three-layer KPI card overlay stack at all (its KPI strip was removed entirely during design) — has by far the fewest total visual objects and the lowest ceiling. This matches Power BI's own documented behavior: visual rendering is single-threaded and sequential per page, so more objects means more queueing regardless of how fast any individual object's own DAX is. Every card's own genuine query cost, where checked, was trivial.

**This is not something to redesign.** The three-visual KPI overlay pattern (base value + colored delta + caption) is this report's established, deliberate construction, used consistently for a real reason (Card (new)'s reference-label conditional formatting remains unreliable). Trading that reliability for fewer objects would reintroduce a known bug to chase a queueing effect that isn't actually a user-facing problem at these durations.

## Status: REOPENED — a severe, separate finding surfaced after initial freeze

Real-world stopwatch testing found page-to-page navigation taking 5–12 seconds — far beyond anything the in-page "Refresh visuals" testing above could have caught, since that action only re-runs visuals already loaded on a page currently open. It never measures the cost of navigating *to* a page.

**A dedicated navigation-capture export (all six pages, five real page-changes) confirms this is real, severe, and has a specific, previously-undiscovered cause:**

| Destination | Gap before the first visual even starts rendering | Total visual objects on that page |
|---|---|---|
| Page 1 | 14.51s | 63 |
| Page 6 | 13.45s | 57 |
| Page 4 | 12.99s | 57 |
| Page 2 | 9.73s | 40 |
| Page 3 | 9.48s | 40 |

**Performance Analyzer logs zero events of any kind during this gap** — not DAX, not rendering, not even parameter resolution. Once the gap ends, all of a page's visuals then load in a fast, heavily-overlapping burst (Page 1's 63 visuals: 5.3 seconds total, not the sequential one-by-one queueing the original "Card count" theory below assumed).

**Correction to the earlier "Card count / single-threaded queueing" finding in this document:** that explanation was based on in-page refresh timing only and does not explain this gap — it may still be a real, minor effect during an in-page refresh, but it is not the cause of the much larger navigation delay reported here. This is a different bottleneck, in a different, uninstrumented layer of Power BI (most likely bookmark application or page-transition overhead, not visual rendering).

**Open question, blocking diagnosis:** what Action Type is actually configured on this report's navigation buttons — Page navigation, or Bookmark? Step 3's documentation assumed simple Page navigation actions with bookmarks reserved only for filter flyouts; if navigation is actually bookmark-driven, that's the likely direct cause, especially if that bookmark's scope is broader than intended.

**Investigation findings:**
- Confirmed via Action Type: navigation buttons use plain **Page navigation**, not a bookmark — the leading hypothesis from Step 3 is ruled out as the cause.
- Confirmed via a blank-page test: navigating to an empty page is near-instant regardless of which UI element triggers it — **the delay is driven by the destination page's visual-object count, not the navigation mechanism.**
- Live memory reading during general use: 16GB total RAM, 84% system-wide utilization (~13.4GB in use), Power BI Desktop alone accounting for 6.9GB — leaving roughly 2.6GB free headroom. This is a plausible, real contributor (first-time visual construction requires fresh memory allocation, work that sits below Performance Analyzer's instrumentation), though not confirmed with a controlled before/after test.

**Decision: accepted as-is, not pursued further.** The file totals **14 pages — 6 dashboard pages plus 8 tooltip pages** — a substantial report by any measure. A 9.5–14.5s worst-case one-time page load in that context, on a machine already running other applications alongside Power BI Desktop, is treated as a reasonable real-world result rather than a defect requiring further engineering. Noted as a specific practice for future builds: budget memory headroom and page/visual-object count more deliberately from the start, rather than discovering the cost at the end.

**Status: Frozen.**
