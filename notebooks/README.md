# Hospital Readmissions Analysis — CMS FY 2026 Data

An analysis of U.S. hospital readmission performance using public CMS (Centers for Medicare & Medicaid Services) data, exploring how readmission rates vary by state, hospital quality rating, medical condition, and ownership type.

## Question

Hospital readmissions are costly and often preventable. This project asks: **what actually predicts whether a hospital has higher or lower readmission rates than expected?**

## Data Sources

- **[Hospital Readmissions Reduction Program](https://data.cms.gov/provider-data/)** (FY 2026) — CMS's risk-adjusted "Excess Readmission Ratio" per hospital, per condition
- **Hospital General Information** (FY 2026) — hospital-level metadata: overall star rating, ownership type, location

Both are public datasets from CMS's Care Compare program, downloaded as CSVs.

**What "Excess Readmission Ratio" means:** it's CMS's own risk-adjusted metric — predicted 30-day readmissions divided by expected readmissions, based on hospitals with similar patient populations. A ratio above 1.0 means a hospital readmits more patients than expected for its case mix; below 1.0 means fewer. Because it's already risk-adjusted, it's usable for direct cross-hospital comparison.

## Approach

1. Loaded both CSVs into a local **SQLite database**
2. Cleaned CMS's text sentinel values (`"Too Few to Report"`, `"N/A"`, `"Not Available"`) into proper nulls, and mapped condition codes to readable labels
3. Wrote **SQL queries** (joins, aggregations, variance calculations) to answer four questions
4. Visualized the key findings with **matplotlib**, using a consistent, colorblind-safe palette
5. Documented data caveats and limitations throughout

**Tools:** Python, pandas, SQLite (`sqlite3`), matplotlib

## Key Findings

- **Geography matters, but modestly.** Massachusetts, New Jersey, and Florida have the highest average excess readmission ratios (~1.02–1.03); Idaho, North Dakota, and Maine have the lowest (~0.94–0.95) — a real but not dramatic spread.
- **Star rating correlates with readmission performance.** There's a clean, monotonic relationship: 1-star hospitals average 1.045, 5-star hospitals average 0.964 (r ≈ -0.44). Higher-rated hospitals do tend to have fewer excess readmissions.
- **Hip/Knee Replacement is far more variable than other conditions.** Its coefficient of variation (0.155) is roughly 2–3x higher than the other five tracked conditions (0.045–0.096) — consistent with it being an elective procedure where surgical approach and rehab protocols vary widely by hospital.
- **Ownership type has a real but small effect.** For-profit hospitals average the worst readmission performance (1.013) of the three major ownership categories; nonprofit and government hospitals are close to par (~1.00).

## Limitations & Caveats

- All averages are **unweighted** across hospitals — a hospital with 20 discharges counts the same as one with 2,000. A discharge-volume-weighted analysis could shift some rankings and is a natural next step.
- About **36% of readmission records** (~6,600 of 18,330) have no usable ratio, mostly because CMS suppresses small counts (`"Too Few to Report"`) to protect patient privacy. These are excluded from all aggregates.
- **10 facility IDs** in the readmissions file have no match in the hospital info file (out of 3,055 total) and are dropped by the join — a negligible 0.3% of hospitals.
- The **"Other" ownership category** (Veterans Health Administration, Department of Defense, Tribal, physician-owned) is a small sample (60 hospitals) — its result shouldn't be over-interpreted.

## Running This Notebook

```bash
pip install pandas matplotlib jupyter --break-system-packages
jupyter notebook notebooks/hospital_readmissions_analysis.ipynb
```

The notebook builds `hospital_readmissions.db` from the CSVs on first run and can be re-executed top to bottom.
