# Hospital Quality and Readmissions Analysis

> Which US hospitals have higher-than-expected 30-day readmissions, what hospital characteristics are associated with that, and how well can patient-level data predict who gets readmitted?

![Dashboard preview](docs/images/dashboard_overview.png)

**Author:** Christian | [LinkedIn](https://linkedin.com/in/your-handle) | [Portfolio](https://your-site.com)
**Tools:** SQL (SQLite) · Python (pandas, scikit-learn, statsmodels) · Power BI · Excel
**Status:** Complete / In progress *(edit this line)*

> **Before publishing:** replace every `[bracketed placeholder]` with your real results. Do not publish placeholder numbers.

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Business Questions](#business-questions)
3. [Key Findings](#key-findings)
4. [Recommendations](#recommendations)
5. [Data Sources](#data-sources)
6. [Repository Structure](#repository-structure)
7. [How to Reproduce](#how-to-reproduce)
8. [Methodology](#methodology)
9. [Data Cleaning Decisions](#data-cleaning-decisions)
10. [Model Results](#model-results)
11. [Dashboard](#dashboard)
12. [Limitations](#limitations)
13. [Ethics and Privacy](#ethics-and-privacy)
14. [Future Work](#future-work)
15. [License and Acknowledgments](#license-and-acknowledgments)

---

## Project Overview

Hospital readmissions are a core quality and cost measure in US healthcare. Medicare penalizes hospitals with excess readmissions through the Hospital Readmissions Reduction Program (HRRP), and readmissions often signal gaps in discharge planning, follow-up care or care coordination.

This project has two parts:

1. **Hospital-level analysis (CMS data):** compare hospitals on readmissions, mortality, infections and patient experience, and examine which hospital characteristics (ownership, type, state, size) are associated with performance.
2. **Patient-level model (UCI diabetes data):** build and evaluate a model that identifies patients at higher risk of 30-day readmission, evaluated in terms a care-management team could act on.

The goal is the same as in any industry: **better decisions that improve care while controlling costs.**

## Business Questions

1. What share of hospitals have readmission rates above expected, by condition?
2. How do readmissions differ by ownership type, hospital type and state?
3. Are readmissions related to patient experience scores and hospital-acquired infection rates?
4. Which hospitals underperform on several measures at once?
5. At a realistic outreach capacity, how well can a model identify patients likely to be readmitted within 30 days?

## Key Findings

> Replace the placeholders with real numbers after running the analysis. Keep each finding to one sentence with a number.

- **[X]%** of hospitals with enough volume had an excess readmission ratio above 1.0 for **[condition]**.
- Hospitals with **[ownership type]** ownership had **[higher/lower]** average excess readmission ratios than **[other type]** (**[value]** vs **[value]**).
- Readmission ratio had a **[weak/moderate]** **[positive/negative]** association with patient experience (correlation **[r]**).
- **[N]** hospitals were above 1.0 on **[two or more]** conditions simultaneously.
- The best patient-level model reached an AUC of **[value]**. Targeting the top 10% highest-risk patients captured **[X]%** of readmissions (**[X]x** lift over random).

## Recommendations

> Edit to match your findings.

1. **Target discharge follow-up programs** at hospitals above 1.0 on multiple conditions, since they are the most likely to benefit.
2. **Use a risk score to prioritize outreach** when care-management capacity is limited, focusing on the top risk decile.
3. **Track readmissions alongside patient experience** instead of treating them as separate programs.
4. **Report rates with volume and uncertainty**, so small hospitals are not ranked as best or worst on noise.

**Expected impact and tradeoffs:** *[One or two sentences: cost of outreach vs. avoided readmissions, with assumptions stated.]*

## Data Sources

| Source | Description | Used for | Date pulled |
|---|---|---|---|
| [CMS Provider Data Catalog](https://data.cms.gov/provider-data) | Hospital General Information | Hospital characteristics | [date] |
| CMS Provider Data Catalog | Hospital Readmissions Reduction Program | Excess readmission ratios | [date] |
| CMS Provider Data Catalog | Unplanned Hospital Visits | Readmission and return measures | [date] |
| CMS Provider Data Catalog | Complications and Deaths | 30-day mortality | [date] |
| CMS Provider Data Catalog | HCAHPS (patient survey) | Patient experience | [date] |
| CMS Provider Data Catalog | Healthcare Associated Infections | Infection ratios | [date] |
| [UCI ML Repository](https://archive.ics.uci.edu/) | Diabetes 130-US hospitals (1999-2008) | Patient-level readmission model | [date] |

**Raw data is not stored in this repository.** See [How to Reproduce](#how-to-reproduce) for download steps. Check each source's terms of use before redistributing.

## Repository Structure

```
hospital-quality-readmissions/
├── README.md
├── requirements.txt
├── .gitignore
├── data/
│   ├── raw/                  # downloaded files (not committed)
│   └── clean/                # cleaned outputs and SQLite database
├── sql/
│   ├── 01_master_table.sql
│   ├── 02_join_checks.sql
│   └── 03_analysis_queries.sql
├── notebooks/
│   ├── 01_data_profiling.ipynb
│   ├── 02_cleaning_and_joins.ipynb
│   ├── 03_eda_hospital_level.ipynb
│   └── 04_readmission_model.ipynb
├── dashboard/
│   ├── hospital_quality.pbix
│   └── hospital_master.csv
└── docs/
    ├── data_log.md
    ├── data_dictionary.md
    ├── executive_summary.pdf
    └── images/
```

*Adjust file names to match your actual repo.*

## How to Reproduce

### 1. Clone and set up the environment

```bash
git clone https://github.com/your-username/hospital-quality-readmissions.git
cd hospital-quality-readmissions
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

### 2. Download the data

1. From the CMS Provider Data Catalog (Hospitals topic), download the six CMS files listed above and save them to `data/raw/`.
2. From the UCI Machine Learning Repository, download `diabetic_data.csv` and `IDS_mapping.csv` and save them to `data/raw/`.
3. Record file names, source URLs and download dates in `docs/data_log.md`.

CMS renames files and columns between releases. If a column name differs from the notebooks, check the data dictionary supplied with that file.

### 3. Run in order

1. `notebooks/01_data_profiling.ipynb`
2. `notebooks/02_cleaning_and_joins.ipynb` (creates `data/clean/hospitals.db`)
3. Run the scripts in `sql/` against the database
4. `notebooks/03_eda_hospital_level.ipynb`
5. `notebooks/04_readmission_model.ipynb`
6. Open `dashboard/hospital_quality.pbix` in Power BI Desktop (Windows) and refresh the data source path

### Requirements (`requirements.txt`)

```
pandas
numpy
matplotlib
seaborn
scikit-learn
statsmodels
jupyter
```

## Methodology

### Part 1: Hospital-level analysis

1. **Profile each file:** rows, columns, key, missingness, placeholder values.
2. **Standardize keys:** Facility ID converted to a 6-character string to preserve leading zeros.
3. **Convert placeholders** (such as "Not Available" or "Too Few Cases") to true missing values, with a suppression flag.
4. **Reshape** HRRP from one row per hospital-condition to one row per hospital.
5. **Join** all hospital files into one master table with SQL LEFT JOINs from the hospital information table.
6. **Validate joins:** row counts before and after, duplicate key check, coverage per measure.
7. **Explore:** distributions, group comparisons by ownership, type and state, correlations with patient experience and infections.
8. **Handle small volumes:** a minimum-discharge rule of **[N]** discharges per condition, with results shown with and without the rule.
9. **Composite score (optional):** standardized measures combined with explicit weights, with a sensitivity test on weights.

### Part 2: Patient-level model

1. **Target:** readmitted within 30 days (`readmitted == "<30"`).
2. **Exclusions:** patients discharged to hospice or who died, since they cannot be readmitted.
3. **One encounter per patient** to avoid leakage between training and test sets.
4. **Features:** age band, length of stay, prior visits, number of medications and diagnoses, discharge destination, primary diagnosis group, lab and medication indicators.
5. **Split:** stratified 80/20 train/test split (the positive class is about 9 to 11%).
6. **Models:** logistic regression baseline, then gradient boosting.
7. **Evaluation:** AUC, precision and recall at a top-decile threshold, lift, calibration, feature importance.
8. **Subgroup check:** performance compared across age, gender and race groups.

## Data Cleaning Decisions

| Issue | Decision | Why |
|---|---|---|
| Facility IDs lost leading zeros | Read as text and pad to 6 characters | Prevents failed joins |
| "Not Available" / "Too Few Cases" in numeric columns | Converted to missing, added suppression flag | Placeholders are not values |
| Hospitals with very few discharges | Minimum volume rule of **[N]** | Unstable rates |
| Heavily missing columns in UCI data (for example weight, payer code) | Dropped | Too few usable values |
| Multiple encounters per patient | Kept first encounter | Avoids leakage |
| Died / hospice discharges | Excluded | Cannot be readmitted |
| High-cardinality diagnosis codes | Grouped into broad categories | Reduces sparsity |

*Add any other decisions you made.*

## Model Results

| Model | AUC | Precision (top 10%) | Recall (top 10%) | Lift (top 10%) |
|---|---|---|---|---|
| Logistic regression | [ ] | [ ] | [ ] | [ ] |
| Gradient boosting | [ ] | [ ] | [ ] | [ ] |
| Random selection (baseline) | 0.50 | about [base rate] | about 10% | 1.0x |

**Interpretation:** *[Two or three sentences in plain language. Readmission is hard to predict, so modest performance is expected; explain what the model can and cannot do in practice.]*

**Top drivers:** *[List the 3 to 5 most important features and what they suggest.]*

## Dashboard

The Power BI report has three pages:

1. **Overview:** KPI cards, state map, filters for ownership, hospital type and condition
2. **Peer comparison:** highest excess-readmission hospitals (with volume filter), readmissions vs. patient experience
3. **Hospital drill-down:** one hospital compared with state and national benchmarks

| Overview | Peer comparison | Drill-down |
|---|---|---|
| ![Overview](docs/images/page1.png) | ![Peers](docs/images/page2.png) | ![Drill-down](docs/images/page3.png) |

## Limitations

- **Association is not causation.** Differences by ownership or hospital type may reflect case mix, location or resources.
- **CMS measures are risk-adjusted by CMS; the UCI data is not.** The two parts of this project answer related but different questions.
- **Suppressed data:** small hospitals have missing values, so the analysis under-represents them.
- **UCI data is old and narrow:** diabetic encounters from 130 hospitals, 1999 to 2008. The model should not be applied to current populations.
- **No cost data in this project:** impact estimates rely on stated assumptions.
- **Release dates matter:** CMS refreshes data regularly, so results can change with newer files.

## Ethics and Privacy

- All data used is publicly available and de-identified. No patient-identifiable data is included.
- Hospital rankings can affect reputations. Findings are presented with volume context and uncertainty and are meant to prompt investigation, not to label hospitals as good or bad.
- A readmission risk model could disadvantage patient groups if used carelessly. Subgroup performance was checked, and any real-world use would need clinical oversight and ongoing fairness monitoring.

## Future Work

- Add multiple CMS release years for a trend analysis
- Add cost and payment data (for example Medicare inpatient payments) to estimate the financial impact
- Use a more recent, richer patient-level dataset (for example MIMIC-IV, with credentialed access)
- Fit a mixed-effects model with hospital-level random effects
- Add confidence intervals or shrinkage for small-volume hospitals

## License and Acknowledgments

- Code in this repository: MIT License *(or your choice; add a LICENSE file)*
- Data: CMS Provider Data Catalog and the UCI Machine Learning Repository. Please follow their terms of use.
- Diabetes dataset: Strack et al., "Impact of HbA1c Measurement on Hospital Readmission Rates: Analysis of 70,000 Clinical Database Patient Records," *BioMed Research International*, 2014.

---

*Questions or feedback? Open an issue or reach out via [LinkedIn](https://linkedin.com/in/your-handle).*

