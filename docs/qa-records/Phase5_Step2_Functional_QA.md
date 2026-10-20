# Phase 5 — Step 2: Functional and Analytical QA

SaaS Growth & Unit Economics · Step 1 (DAX/Measure Validation) frozen · No DAX modified in this step

---

## Part A — Cross-Page Reconciliation Matrix (new work — no single page's SOP checks this)

Every number below should match wherever it repeats, at the same filter state (full period, Segment=All, no drill-through). Set every page to that baseline before checking.

| Figure | Where it should appear | Confirmed value | Status |
|---|---|---|---|
| MRR | P1 Tier 1, P2 KPI strip, P5 V3 total row | $1,002.7K | **[CONFIRMED]** — P5's V3 total row already screenshotted at exactly this value against P1/P3 |
| Active Customers | P1 Tier 1, P3 KPI strip, P5 V3 total row | 3,587 | **[CONFIRMED]** — same screenshot |
| ARPA | P1 Tier 2, P4 input card, P5 V3 total row | $279.5 | **[CONFIRMED]** — same screenshot |
| ARR | P1 Tier 1, P2 KPI strip | $12,032.8K | [CHECK] — not yet cross-confirmed live in this project |
| Logo Churn Rate | P1 Tier 2, P3 KPI strip, P6 K3 | 2.40% | [CHECK] |
| NRR | P1 (Insight card text), P6 K1 | 99.06% | [CHECK] |
| GRR | P6 K2 | 94.5% | **[CONFIRMED]** — Step 1, three independent derivations agreed |
| New Customers | P1 (waterfall), P3 KPI strip | 177 | [CHECK] |
| Churned Customers | P1 (waterfall), P3 KPI strip | 84 | [CHECK] |
| Net MRR Change | P1 (Insight card), P2 KPI strip | $29,322.38 | [CHECK] |

**For each [CHECK] row:** open both pages side by side (or two browser/app windows), confirm the figure matches to the displayed precision. Any mismatch here is high-priority — it means either a stale measure, a filter accidentally left applied on one page, or a genuine unresolved duplicate like the CAC one just found in Step 1.

## Part B — Analytical Sanity Checks (relationships between numbers, not just repeated ones)

These aren't places the same number appears twice — they're places two *different* numbers should have a provable mathematical relationship. This is a stronger test than reconciliation, because a bug that produces a plausible-looking wrong number in isolation often fails this kind of cross-check.

| Check | Expected relationship | Where to verify | Status |
|---|---|---|---|
| New − Churned = Net Change | 177 − 84 = 93 | P3 KPI strip vs. Insight-adjacent figure | [CHECK] |
| New Rate − Churn Rate = Customer Growth % | Already proven exactly, via the July 2024 churn-wave residual test | P3 — no action, already validated | Closed |
| GRR + Revenue Churn Rate ≠ 100% (by design) | 94.53% + 1.30% = 95.83% | P6 — already confirmed in Step 1 | Closed |
| LTV ÷ CAC = LTV:CAC ratio | Displayed ratio should equal LTV divided by CAC, on every channel row | P4 Channel Detail table | [CHECK] — flagged in the P4 SOP's own QA section, confirm it was actually run |
| MRR ≈ Active Customers × ARPA | Should hold approximately on P5's V1 scatter (this is the whole premise of using MRR as bubble size) | P5 V1, spot-check 2–3 bubbles | [CHECK] |
| Plan shares sum to 100%, at baseline and within any selected group | Confirmed at baseline (Step 1) and within Professional Services (100.0% exactly) | P5 V2 | Closed |

## Part C — Per-Page Final QA (already written — this is the "go run it" reminder, not new content)

Each page's SOP ends in its own Final QA / Checkpoint section. Given Step 1 changed real numbers on Pages 1, 3, 4, and 6, re-running these now — not trusting a pass that happened before those fixes — is the point of this step.

| Page | What changed since it was last checked | Action |
|---|---|---|
| P1 | `_Active Customers Net Change` was deleted and restored | Re-confirm the Active Customers caption reads correctly |
| P3 | Same measure, same caption | Re-confirm |
| P4 | Orphaned `Blended Marketing CAC` deleted | Confirm the Channel Economics scatter and CAC KPI card are unaffected (they were already using the correct `(P4)` version, so this should be a non-event — confirm it actually is) |
| P6 | 4-card KPI strip built, Revenue Churn Rate added | Run the full Section 11 Final QA in the Page 6 SOP — this page has the most unverified surface area of any page going into this step |
| P2, P5 | No direct changes this step | Standard pass — confirm nothing regressed as a side effect of work on other pages |

## What Closes Step 2

All [CHECK] rows in Parts A and B resolved, plus a confirmation that Part C's per-page pass was actually run on all six pages — not assumed clean because nothing was touched directly.
