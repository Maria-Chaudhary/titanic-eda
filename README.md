# Exploratory Data Analysis – Titanic Passenger Dataset

Task 1 (EDA): ingest a public dataset, profile and clean it with justified decisions, visualise it, and document practical takeaways under every figure.

**Dataset:** Titanic passenger manifest, 891 rows × 15 columns, from the public [`seaborn-data`](https://github.com/mwaskom/seaborn-data) repository (`titanic.csv`).

## Deliverables

| Item | Location |
|---|---|
| Fully executed notebook with markdown commentary | `EDA_Titanic_Analysis.ipynb` |
| High-resolution figures (300 dpi PNG, 9 files) | `images/` |
| Raw and cleaned data | `data/titanic.csv`, `data/titanic_clean.csv` |
| This summary | `README.md` |

## How to run

```bash
pip install -r requirements.txt
jupyter notebook EDA_Titanic_Analysis.ipynb      # Kernel > Restart & Run All
```
The notebook downloads the CSV itself if `data/titanic.csv` is missing. All random steps use `SEED = 42`.

## Data characteristics

* **Shape:** 891 × 15 raw → **784 × 12** cleaned, zero missing values.
* **Missing data:** `deck` 77.2 %, `age` 19.9 %, `embarked` 0.2 %. Age missingness is *not* random: 27.7 % in 3rd class vs 6.0 % in 2nd (χ² = 46.1, p < 0.001).
* **Duplicates:** 107 exact duplicate rows (12.0 %).
* **Skew / outliers:** `fare` skew 4.57 (mean £34.71 vs median £15.90); 102 rows (13.0 %) above the IQR fence of £73.20.
* **Outcome balance:** 41.2 % survived, 58.8 % died (after cleaning).

## Cleaning decisions and justification

| Issue | Decision | Evidence |
|---|---|---|
| 107 exact duplicates | **Dropped** (as the brief requires) | No ID column; duplicates are 80 % 3rd class vs 55 % overall, so some may be distinct passengers. Sensitivity check: survival rate 38.4 % → 41.2 %, corr(sex, survived) 0.543 → 0.516, so conclusions are unchanged. |
| `deck` (74 % missing after dedup) | **Dropped** | Only 202 observed values; missing by class (known for 81 % of 1st class vs 2.4 % of 3rd), so imputing would invent data. |
| 6 redundant columns (`class`, `alive`, `embark_town`, `alone`, `who`, `adult_male`) | **Dropped** | Proven one-to-one or derivable from other columns. |
| `age` (13.5 % missing after dedup) | **Imputed**: median by `pclass × sex`, flag `age_imputed` kept | η² = 0.165 for class × sex. In 200 random 20 % masking trials RMSE fell from 14.71 (global mean) to 13.55 (−7.9 %). Std shrinks 14.76 → 13.95; KS D = 0.039, p = 0.62. |
| `embarked` (2 rows) | **Mode imputed** (S, 72.4 %) | Worst-case effect on any port share: 0.26 pp. |
| `fare` outliers | **Kept**, added `fare_log = log1p(fare)` | Skew 4.57 → 0.42; Pearson r with survival 0.247 → 0.313 (equals Spearman 0.308). Dropping the outliers lowers r to 0.199 and removes 102 mostly 1st-class rows with 72.5 % survival. |
| `age`, `sibsp`, `parch` extremes | **Kept** | Plausible values (age 0.42–80; large families). |

## Key analytical insights

1. **Sex is the strongest signal.** Women survived at 74.1 % vs 21.6 % for men (Cramér's V = 0.51; Pearson r = +0.52).
2. **Class matters within each sex.** Women: 97 % / 92 % / 47 % in classes 1 / 2 / 3. Men: 37 % / 18 % / 16 %. A 3rd-class woman had better odds than a 1st-class man.
3. **Age alone is weak** (r = −0.075; survivors vs victims median 28 vs 28, p = 0.156), yet it hides a **child effect**: 57.4 % survival at ages 0–12 vs 34.4 % at 51+ (observed ages only).
4. **Family size is non-linear.** Solo travellers 33.6 %, family of 2–4 55–71 %, family of 5+ only 18.2 % (47 of those 55 passengers were 3rd class). Pearson r ≈ 0.02 but Spearman ρ = 0.14 reveals this.
5. **Fare is informative but redundant with class.** Survivors paid a higher median fare (£26.2 vs £13.0), but corr(log fare, class) = −0.69, so both should not be fed blindly into a linear model.
6. **Port of embarkation is confounded by class** (Queenstown 91.5 % 3rd class; Cherbourg 53.5 % 1st class), so it should not be read causally.

## Figures (`images/`)

| File | Content |
|---|---|
| `fig01_missing_values.png` | Missing-value bars and missingness map |
| `fig02_age_imputation.png` | Imputation benchmark and before/after age distribution |
| `fig03_fare_outliers_log_transform.png` | Fare histogram and boxplot, raw vs log |
| `fig04_numeric_distributions.png` | Feature distributions with mean/median lines |
| `fig05_categorical_distributions.png` | Counts and shares for outcome, class, sex, port |
| `fig06_boxplots.png` | Age and fare boxplots by class, sex, port and outcome |
| `fig07_correlation_heatmaps.png` | Pearson and Spearman heatmaps |
| `fig08_pairplot.png` | Pairwise scatter plots coloured by outcome |
| `fig09_survival_segments.png` | Survival by sex × class, age band, family size (95 % Wilson CIs) |

## Limitations

* Findings are associations from one historical event, not causal effects.
* Duplicate removal is a judgement call without a passenger ID (see table above).
* The age imputer is modest (7.9 % RMSE gain) and slightly compresses variance.
* Small segments (e.g. family size 4, n = 28; Queenstown, n = 59) have wide uncertainty.
