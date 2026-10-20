# Phase 5 — Step 7: PBIT / Template Readiness

SaaS Growth & Unit Economics

---

## Core Finding

A `.pbit`'s "prompt for values on open" behavior is specific to **Power Query parameters** — a separate mechanism from the **What-if parameters** already built throughout this project (Assumed Gross Margin %, LTV:CAC Target, Min Volume). This project had never built a Power Query parameter. Every table's Source step referenced a hardcoded file path. **Exporting to `.pbit` before this fix would not have given a new user anything to plug their own data into** — the single most important promise of "reusable template" wouldn't have been deliverable, despite every other reusability decision across Steps 1–6 being genuinely sound.

## Work Completed

- Built `SourceFilePath` as a Power Query parameter (Text type).
- Repointed every table's Source step (`DimCustomer`, `DimPlan`, `DimDate`, `FactSubscription`, `FactSubscriptionEvent`, `FactMarketing`, `FactCustomerHealth`) from a hardcoded path to `Excel.Workbook(File.Contents(SourceFilePath), null, true)`.
- **Live-tested the actual capability this exists for:** changed `SourceFilePath` to a different valid file location, confirmed the model refreshed correctly against it, confirmed every table loaded. This is the first time in the project this specific claim was verified end-to-end rather than assumed.
- Confirmed all three What-if parameters (Gross Margin, LTV:CAC Target, Min Volume) retained their correct default values throughout.

## Incident, Root Cause, and Resolution — Worth Recording in Full

**Symptom:** after the parameterization work, Page 1's MRR Movement waterfall changed shape and its numbers changed. Separately, Page 4 went almost entirely blank (CAC, LTV:CAC, Payback, the scatter, Where Spend Sits, most of Channel Detail) while LTV and ARPA remained correct.

**What was ruled out, in order, each with a real check rather than assumption:** Page 4 slicer/parameter state, the Filters pane, a stuck visual selection, `FactMarketing`'s Enable Load setting, the `FactMarketing ↔ DimAcquisitionChannel` and `FactMarketing ↔ DimDate` relationships (both confirmed present, active, and correctly configured), and column data types on both sides of each relationship.

**Actual root cause:** during the earlier test where `SourceFilePath` was deliberately changed to a different path to prove the mechanism worked, the value was not fully reverted to the original file — it was pointed at a similar but not identical file (most likely the duplicated backup copy in a different folder). The model was therefore genuinely, successfully loading a *different* file than before — not the same data through a broken mechanism. This explains both symptoms as one cause: if `FactMarketing`'s data differs even slightly between the two files, that alone accounts for Page 4 going blank; a different underlying dataset also explains the waterfall's changed shape and numbers on Page 1.

**Resolution:** compared `SourceFilePath`'s current value character-by-character against the original file's actual path (read directly from an untouched backup copy's own Source step). Corrected the mismatch, Close & Apply, confirmed both Page 1 and Page 4 returned to their correct, original state.

**Standing lesson:** verifying the parameter mechanism *works* (Step 7's original goal) required actually changing its value — but that test step itself introduced real risk if the revert isn't exact. Worth remembering for any future template work: a test that proves reusability by deliberately changing a live setting needs its own careful, verified revert, not just "set it back" from memory.

## Target/Benchmark Layer — Decision

**Confirmed by explicit user decision: Option B — deferred, not built.** The schema (MetricKey, MetricDisplayName, TargetValue, Direction, ToleranceBand, BenchmarkType, EffectiveFrom, EffectiveTo, SourceNote) remains designed but unbuilt. Reasoning: the table alone would be inert — no measures compare actuals to targets, no visual reads from it, and building only the schema this late in Phase 5 would be a shallow, disconnected addition rather than a real capability. This is real future work (new measures, a `Direction`-aware color rule, specific visual placement decisions) deserving its own design pass, not a checklist item. Documented here for Step 8 and for whoever picks this up next.

## Placeholder Naming — Confirmed Clean

"[Company Name]" confirmed identical on every page's header, a plain literal text string — not bound to anything, obviously editable by a new user. No stray company-specific branding found in channel/plan labels (those are the dataset's genuine category names). No leftover scaffolding text found.

## Export — Completed and Verified End-to-End

Completed ahead of the originally planned sequencing (after Step 8) — a deliberate, acknowledged trade-off: if Step 8's documentation work leads to any change inside the file itself, one further re-export would be needed.

1. **File → Export → Power BI template**, with a fully generic description (no company-specific reference) stating the expected file structure.
2. **File size confirmed dramatically smaller** than the `.pbix` — data excluded, structure retained.
3. **The decisive test: closed the working file entirely, double-clicked the exported `.pbit` fresh** — the way a genuinely new user would receive it. **Confirmed: the parameter prompt appeared with the written description, `SourceFilePath` was entered, and the report loaded fully populated**, verified on Page 1's MRR card and Page 4's CAC card.

**This is the first point in the project where "reusable template" was proven experientially, not just inferred from correct DAX or a passed filter test.**

## Status: FROZEN — complete.
