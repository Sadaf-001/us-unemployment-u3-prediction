# Predicting the U-3 Unemployment Rate

Linear regression with forward selection on FRED labor-market and macro indicators, built to estimate the U.S. U-3 unemployment rate for December 2025.

Group project for *Statistical Models for Data Science* (Analysis Challenge), University of Chicago.
**Group 8:** Devanshu Khadka, Mahima Goyal, Jiayi Hui, Sadaf Khan. Instructor: Prof. Ming-Long Lam.

## Approach

1. **Data.** Seven monthly series from FRED (UNRATE as target; JTSJOL, ICSA, CCSA, CPIAUCSL, FEDFUNDS, PAYEMS as inputs). Weekly claims were averaged to monthly. December 2000 to August 2025.
2. **Feature engineering** (`EDA.ipynb`). Log transforms of claims, 1- and 3-month lags, month-over-month percent changes, and 3-month rolling averages. This gives 20 candidate predictors.
3. **Model selection** (`Analysis_challnege_modelling_forward_selection_and_linear_regression.ipynb`). Forward selection using F-tests (entry threshold p = 0.05) on 294 monthly observations, ending with 10 predictors:
   `UNRATE_ma3, PAYEMS_pct, ICSA_lag1, CCSA_ma3, CCSA, ICSA_ma3, log_ICSA, PAYEMS, ICSA_lag3, PAYEMS_lag1`
4. **Final model and evaluation** (`Model_Evaluation.ipynb`). OLS fit on the full history, with fit metrics and a residual plot.

## Data sources (all public, from FRED)

| Series | FRED ID | Frequency | Role |
|---|---|---|---|
| U-3 unemployment rate | UNRATE | Monthly, SA | Target |
| Job openings | JTSJOL | Monthly, SA | Labor demand |
| Initial claims | ICSA | Weekly, SA, averaged to monthly | Layoff signal |
| Continued claims | CCSA | Weekly, SA, averaged to monthly | Persistence of unemployment |
| CPI, all urban consumers | CPIAUCSL | Monthly, SA | Inflation |
| Effective federal funds rate | FEDFUNDS | Monthly | Monetary stance |
| Total nonfarm payrolls | PAYEMS | Monthly, SA | Employment growth |

Find any series at `https://fred.stlouisfed.org/series/<ID>`. The data files themselves are not stored in this repo.

## Results (in-sample)

| Metric | Value |
|---|---|
| R² | 0.9977 |
| Adjusted R² | 0.9976 |
| RMSE | 0.0945 |
| MAE | 0.0711 |
| Residual variance | 0.0093 |

Model estimate from the final row of the dataset: **4.23%** (presented as the December 2025 estimate).

## Limitations and next steps

These matter when reading the numbers above:

- **No out-of-sample test.** All metrics are computed on the same data the model was fit and selected on. A time-based holdout or rolling-origin evaluation is the natural next step.
- **Target leakage.** `UNRATE_ma3` is a 3-month average that includes the current month's unemployment rate, i.e. the target. It is the first variable forward selection picks (coefficient about 0.995), and it accounts for much of the very high R².
- **Same-month predictors.** Most predictors are measured in the same month as the target, so the model works as a nowcast. A true forecast needs inputs that are lagged so they are known before the target month is released.
- **Multicollinearity.** VIFs are extreme for `PAYEMS` and `PAYEMS_lag1` (both above 200,000), so individual coefficients are not reliable to interpret.
- **COVID-19 outliers** were kept in the data and are not modeled separately.

Suggested improvements: drop target-derived features or lag them, compare against a naive "last value" and an AR baseline, and evaluate with a rolling-origin backtest.

## Repository contents

Inside the `Statistical Model for data science` folder:

```
EDA.ipynb                                                                  # loading, EDA, feature engineering
Analysis_challnege_modelling_forward_selection_and_linear_regression.ipynb # forward selection, final model, estimate
Model_Evaluation.ipynb                                                     # fit metrics and residual plot
G8_U3_LaborPrediction_Dec2025.pptx                                         # final presentation
```

## Running it

The notebooks were written in Google Colab.

1. Install dependencies: `pip install numpy pandas scipy statsmodels matplotlib`
2. Provide the input data. `EDA.ipynb` reads `monthly_panel.csv`; the modeling and evaluation notebooks read `/content/unemployment_model_input.csv` (294 rows). Neither file is included here, so you would need to rebuild them from the FRED series above.
3. The modeling and evaluation notebooks import a `Regression` helper module (`import Regression`) provided in the course. It is not included in this repo.
