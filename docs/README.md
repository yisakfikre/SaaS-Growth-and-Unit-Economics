# Documentation index

| Document | What it covers |
| --- | --- |
| **[METRIC_DEFINITIONS.md](METRIC_DEFINITIONS.md)** | Every metric's formula, unit, grain, sign convention and limitations. Start here to understand the numbers |
| **[DATA_DICTIONARY.md](DATA_DICTIONARY.md)** | All 63 columns across the 7 source tables, with types, constraints and business meaning |
| **[ARCHITECTURE.md](ARCHITECTURE.md)** | Why six pages, what each owns, the model, the design system |
| **[VALIDATION.md](VALIDATION.md)** | What was checked, what failed and was corrected, what remains open, what it does not cover |
| **[LIMITATIONS.md](LIMITATIONS.md)** | Analytical limits, structural limits, accepted decisions, deferred work, maintenance requirements |
| **[SETUP.md](SETUP.md)** | Opening the report, running it on your own data, troubleshooting |

## Supporting material

| Folder | What it is |
| --- | --- |
| **[page-sops/](page-sops/)** | The click-by-click build SOP for each of the six pages, revised to describe the final working build. Pages 1–5 are self-contained HTML — download and open in a browser; Page 6 is Markdown and renders on GitHub |
| **[qa-records/](qa-records/)** | The seven production-readiness step records, **unedited working documents** — they contain the dead ends, the incidents and the reversals, not a polished summary. [VALIDATION.md](VALIDATION.md) is the summary |
| **[Metric_Page_Architecture_Control.html](Metric_Page_Architecture_Control.html)** | The full metric-ownership matrix with per-metric reasoning. Self-contained HTML — download and open in a browser |

## ⚠️ Known stale content

Two documents here predate the Phase 5 corrections and are kept as build-history records rather than current references:

- **`Metric_Page_Architecture_Control.html`** cites health-at-churn as 60.02 / 19.62. Those values were tested, failed to reproduce, and were **retired** in favour of **54.71 / 12.91**. Read it for the ownership model, not for figures.
- **`qa-records/Phase5_Step6_Data_Contract_Testing.md`** states that exclude-only Top-N behaviour (no "Other" bucket) was accepted as Page 4's final design. **The shipped build has an "Other" bucket**, produced by the `Channel Group` calculated column. The build is correct; that record is stale.

Both discrepancies are documented in [VALIDATION.md §6](VALIDATION.md#6-open-and-unresolved). Where a document here and the shipped `.pbit` disagree, **the `.pbit` is authoritative.**
