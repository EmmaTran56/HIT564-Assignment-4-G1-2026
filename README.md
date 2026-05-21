# PRT564 — Assessment 4 (Group 1)

End-to-end pipeline for **Northern Territory crime**, **population**, and **wholesale alcohol supply (PAC)**. The main script **cleans and merges** the source files, writes an analysis-ready dataset, then runs:

- **EDA** for **RQ1** (monthly crime patterns) and supporting assault analysis
- **Classification** for **RQ3** (region-month risk) and **RQ4** (seasonal month-level risk)

## Research questions

| RQ | Focus | Approach |
|----|--------|----------|
| **RQ1** | Monthly crime patterns across NT | Descriptive EDA (offences, rates, heatmaps) |
| **RQ3** | Classify **region-months** into High / Medium / Low violent-crime risk | GaussianNB, SVM, Random Forest (train 2024 → test 2025) |
| **RQ4** | Classify **months** for seasonal resource planning | Same models on month-aggregated data (temporal + alcohol features only) |

Risk labels use **tertiles** (33rd / 66th percentiles) of assault rate, with thresholds fit on **2024 only** to avoid leakage into the 2025 test set.

## Requirements

- **Python** 3.8+ recommended
- **Packages**: `pandas`, `numpy`, `matplotlib`, `seaborn`, `openpyxl`, `scikit-learn`, `scipy`, `shap`

Install example:

```bash
python -m pip install pandas numpy matplotlib seaborn openpyxl scikit-learn scipy shap
```

## Input data

Place these files in the **same directory** as the script (paths are relative to the script folder):

| File | Description |
|------|-------------|
| `nt_crime_statistics_dec_2025.csv` | NT crime statistics |
| `nt-government-regions_1986-to-2025.xlsx` | Population by NT Government region |
| `wholesale-alcohol-supply-by-quarter-2023.xlsx` | Alcohol supply (for imputation context) |
| `wholesale-alcohol-supply-by-quarter-2024.xlsx` | Alcohol supply |
| `wholesale-alcohol-supply-by-quarter-2025.xlsx` | Alcohol supply |

## How to run

From this folder:

```bash
python assignment_4.py
```

The script runs **Steps 1–6** (load → merge → save CSV), **feature engineering**, **EDA**, then **classification** for RQ3 and RQ4. Plot folders are created automatically if missing.

## Outputs

| Output | Description |
|--------|-------------|
| `nt_crime_merged.csv` | Merged table: crime counts + population features + PAC + region dummies |
| `eda_plots/*.png` | EDA figures (overview, RQ1, assault analysis) |
| `classification_plots/*.png` | Classification EDA, model comparison, SHAP, and prediction plots |

### Generated plots (`eda_plots/`)

- **Section 1 — Overview:** `1_1_population_by_region.png`, `1_2_offences_by_category.png`, `1_3_alcohol_dv_involvement.png`
- **Section 2 — RQ1 (monthly patterns):** `2_1_offences_by_month.png`, `2_2_crime_rate_per_100k_by_month.png`, `2_3_monthly_trend_2024_vs_2025.png`, `2_4_heatmap_month_category.png`
- **Section 3 — Assault analysis:** `3_1_assault_by_region.png`, `3_2_assault_trend_by_region.png`, `3_3_scatter_pac_vs_assault.png`, `3_4_correlation_heatmap.png`

### Generated plots (`classification_plots/`)

- **Classification EDA:** `CLS1_risk_by_region.png`, `CLS2_risk_by_month.png`, `CLS3_feature_correlation.png`
- **Per RQ (RQ3 and RQ4):** `{RQ}_model_comparison.png`, `{RQ}_confusion_matrices.png`, `{RQ}_rf_feature_importance.png`, `{RQ}_shap_high.png`
- **RQ3 only:** `RQ3_predicted_vs_actual_heatmap.png`
- **RQ4 only:** `RQ4_monthly_predictions.png`

## What the pipeline does (summary)

1. **Crime**
   - Drops `Unknown` reporting region and **year 2023** (only partial year in source).
   - Remaps rows to **six NT Government statistical regions** (`Greater Darwin`, `Central Australia`, `Big Rivers`, `East Arnhem`, `Barkly`, `Top End`).
   - For **NT Balance**, uses **SA2** (`Statistical Area 2`) to assign the correct population region where possible; unmatched SA2 defaults to **Top End**.
   - Encodes **Alcohol involvement** and **DV involvement** as binary **0/1** (`-` → 0).

2. **Population**
   - Filters to **2024–2025**, builds totals, Aboriginal / non-Aboriginal, sex splits, and **age-group** columns (`Pop_age_*`).

3. **Alcohol**
   - Loads **2023–2025** quarters.
   - Maps police-style regions to the same **six** regions; **Darwin + Palmerston → Greater Darwin** and **PAC is summed** for that combined region.

4. **Merge and aggregation**
   - Left-joins crime to alcohol on **Year, Quarter, Region**, then population on **Year, Region**.
   - Aggregates **Number of offences** across consistent dimensions.

5. **PAC imputation**
   - Missing PAC (notably **Q3–Q4 2025** in the source) is filled with the **mean of the same Region × Quarter** across available rows, then rounded to integers.

6. **Modelling helpers**
   - One-hot encodes **Region**; **Greater Darwin** is dropped as the **reference** category.

7. **Classification (RQ3 & RQ4)**
   - Builds a **region-month** dataset (assault rate, demographics, per-capita alcohol, seasonal `sin`/`cos` month).
   - **RQ3:** 16 features including population and demographics; predicts risk for each region in each month.
   - **RQ4:** Aggregates to **12 months × 2 years**; uses only temporal and alcohol features (no static demographics).
   - Tunes **Random Forest** and **SVM** with **GridSearchCV** + **TimeSeriesSplit** (`f1_weighted`); fits **GaussianNB** with a scaler pipeline.
   - Evaluates on **2025** hold-out; reports confusion matrices, weighted metrics, paired **t-test** (GNB vs RF on CV), and **SHAP** for the high-risk class (RF).

## Notes

- If `nt_crime_merged.csv` is open in **Excel** or locked by **OneDrive**, saving may fail with a permission error—close the file or pause sync and rerun.
- Crime analysis years in the merged file are **2024–2025**; alcohol still uses **2023** where needed for imputation and quarter structure.
- **RQ4** uses only **24** month-level observations; treat results as indicative rather than definitive.

## Authors

**PRT564 — Data Analytics and Visualisation**, Assessment 4, **Group 1**.
