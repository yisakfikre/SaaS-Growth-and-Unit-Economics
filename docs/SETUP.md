# Setup guide

Three ways to use this repository, from zero effort to running it on your own data.

---

## Option 1 — Just read it

Nothing to install. The six page images and the documents in this folder contain the complete analysis.

| | |
| --- | --- |
| **[All six pages](../images/)** | ~1790 × 1000 each |
| **[Metric definitions](METRIC_DEFINITIONS.md)** | Every formula, sign convention and limitation |
| **[Validation report](VALIDATION.md)** | What was checked, what failed, what remains open |
| **[Architecture](ARCHITECTURE.md)** | The six-page metric-ownership model |
| **[Data dictionary](DATA_DICTIONARY.md)** | Every field in all seven tables |
| **[Limitations](LIMITATIONS.md)** | Design decisions, deferred work, maintenance requirements |
| **[DAX export](../powerbi/measures.dax)** | All 180 measures, readable without Power BI |

---

## Option 2 — Open the report with the bundled data

**Requires:** Power BI Desktop (free, Windows only) — [powerbi.microsoft.com/desktop](https://powerbi.microsoft.com/desktop/)

1. Download **both** files:
   - `powerbi/SaaS_Growth_Unit_Economics.pbit`
   - `data/SaaS_Growth_Unit_Economics_Dataset.xlsx`
2. Note the full path to the `.xlsx` — e.g. `C:\Users\you\Downloads\SaaS_Growth_Unit_Economics_Dataset.xlsx`. You will need to type or paste it exactly.
3. **Double-click the `.pbit` file.**

   > ⚠️ Open it by double-clicking, **not** through Power BI's *File → Open*. The double-click is what triggers the parameter prompt. Opening it the other way can skip the prompt and leave you with an unloaded model.

4. A dialog appears asking for **`SourceFilePath`**. Paste the full path to the `.xlsx`, including the filename and extension.
5. Click **Load**.
6. Power Query runs, the model refreshes, and all six pages populate. First load takes a minute or two.

### What you should see

| Page | A figure to check |
| --- | --- |
| Executive Overview | MRR **$1.0M**, Active Customers **3,587**, NRR **99.1%** |
| Revenue & Growth | Net MRR Change **$29.3K** |
| Customer Growth | New **177**, Churned **84** |
| Unit Economics | Blended CAC **$415**, LTV:CAC **35.7×** |
| Segment Analysis | Total MRR **$1,002.7K** |
| Retention & Churn | GRR **94.5%**, Logo Churn **2.4%** |

If these match the images in [`../images/`](../images/), the model loaded correctly.

> **Cold page loads take 9.5–14.5 seconds** on the heavier pages the first time you navigate to each. This is a known, investigated characteristic of a 14-page file, not a failed load. See [VALIDATION.md §8](VALIDATION.md#8-performance).

---

## Option 3 — Run it on your own data

The template reads **one Excel workbook with seven named sheets**. Sheet names and column names must match exactly — Power Query binds to them by name.

### Required schema

| Sheet | Required columns |
| --- | --- |
| `DimCustomer` | CustomerID, CustomerName, AcquisitionChannel, CompanySize, Country, CustomerSegment, Industry, Region, SignupDate |
| `DimPlan` | PlanID, PlanName, AnnualPrice, MonthlyPrice, IncludedUsers, PlanTier |
| `DimDate` | Daily grain, standard calendar attributes |
| `FactSubscription` | SubscriptionID, CustomerID, PlanID, StartDate, EndDate, Status, MonthlyPrice, BillingFrequency |
| `FactSubscriptionEvent` | EventID, CustomerID, SubscriptionID, EventDate, EventType, MRRChange, NewMRR, PreviousMRR, NewPlanID, PreviousPlanID, ChurnReason |
| `FactMarketing` | AcquisitionChannel, Date, Leads, NewCustomers, MarketingSpend |
| `FactCustomerHealth` | HealthRecordID, CustomerID, Date, HealthScore, NPS |

Full types, constraints and business meaning: [DATA_DICTIONARY.md](DATA_DICTIONARY.md).

### Non-negotiable content rules

These are not schema requirements — they are assumptions the DAX makes. Breaking one produces wrong numbers rather than an error.

| Rule | Why |
| --- | --- |
| `EventType` must be exactly `New`, `Expansion`, `Contraction`, `Churn` | Measures filter on these literal strings |
| `MRRChange` must be **signed** — negative for Contraction and Churn | MRR is a running total of this column; unsigned values break every revenue measure at once |
| `MRRChange` must be **normalised to monthly** | Annual contracts enter as annual price ÷ 12, never as contract value |
| Every customer must have exactly one `New` event | `Total Customers` counts distinct customers on `New` events |
| `FactMarketing` is at channel × month grain, not customer grain | It joins the model through `AcquisitionChannel` and `Date` only |

### First-run checklist

1. **Load your file** via the `SourceFilePath` prompt (Option 2, steps 3–5).
2. **Set the three Unit Economics parameters on Page 4** before reading anything there. They ship with illustrative defaults, not neutral ones:

   | Parameter | Ships as | Set it to |
   | --- | --- | --- |
   | Assumed Gross Margin % | 80% | Your actual gross margin |
   | LTV:CAC Target | 50× | Your own target — see the warning below |
   | Minimum Volume Threshold | 30 customers | The volume below which a channel isn't worth judging |

   > ⚠️ **The 50× default is not an industry benchmark.** It reflects the scale this project's LTV formula produces on low-churn synthetic data. The conventional 3× rule of thumb applies to a differently-constructed LTV. Read [METRIC_DEFINITIONS.md §6.3](METRIC_DEFINITIONS.md#63-ltv-formula-limits) before setting this.

3. **Replace `[Company Name]`** in the header on all six pages. It is a plain text box on each page — click it and type.
4. **Check the Page 4 Top-N cutoff.** The channel ranking keeps the top 6 by marketing spend. If you have a different number of meaningful channels, the cutoff must be changed in **three places by hand** — the scatter, the detail table, and the Where Spend Sits panel. Nothing synchronises them. See [LIMITATIONS.md](LIMITATIONS.md#maintenance-requirements).
5. **Verify one figure you already know.** Page 1's MRR card is fastest. If it doesn't match your own records, stop and check the sign convention on `MRRChange` before trusting anything else.

### Changing the source file later

**Home → Transform data → Data source settings**, or edit the `SourceFilePath` parameter directly under **Home → Transform data → Edit parameters**.

> ⚠️ Change it carefully, and revert it exactly. During validation, a test that changed this parameter and reverted it to a *similar but not identical* path caused Page 4 to go nearly blank and Page 1's waterfall to change shape — the model was loading a different file, perfectly correctly. It took a character-by-character path comparison to find. See [VALIDATION.md §5.1](VALIDATION.md#51-the-pbit-would-not-have-worked).

---

## Troubleshooting

| Symptom | Cause |
| --- | --- |
| No parameter prompt on open | You opened it via File → Open. Close without saving and double-click the `.pbit` instead |
| "We couldn't find the file" | The path is wrong, or missing the filename/extension. Paste the full path including `.xlsx` |
| A table fails to load | A sheet name doesn't match. Power Query binds by sheet name — check spelling and case against the schema above |
| Pages load but visuals are blank | The model loaded a file whose sheets exist but whose columns don't match. Check column names, then `EventType` values |
| MRR is wildly wrong or negative | `MRRChange` sign convention — Contraction and Churn must be negative |
| MRR looks plausible but churn is zero | `EventType` strings don't match the four expected literals exactly |
| Page 4 blank but LTV and ARPA fine | `FactMarketing` didn't load, or its `AcquisitionChannel` values don't match `DimCustomer`'s |
| 9–15 second page loads | Expected on heavier pages, first navigation only. See [VALIDATION.md §8](VALIDATION.md#8-performance) |
| Plan-level measure returns the unfiltered total | It's missing `USERELATIONSHIP`. See [METRIC_DEFINITIONS.md §7](METRIC_DEFINITIONS.md#7-segment-and-plan-measures) |

---

## Reproducing the dataset

The synthetic data was generated by a seeded Python simulation (numpy seed `42` for the lifecycle simulation, `43` for the cosmetic data-quality injection), documented in [DATA_DICTIONARY.md](DATA_DICTIONARY.md).

**The generator script is not published in this repository.** The dataset is distributed as the finished workbook. This means the figures in the README can be *inspected* in the report but **cannot be independently recomputed** from this repository — a real limitation, stated in [VALIDATION.md §1](VALIDATION.md#1-what-kind-of-validation-this-is).
