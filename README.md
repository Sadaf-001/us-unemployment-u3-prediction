# Predicting the U-3 Unemployment Rate

Linear regression with forward selection on FRED labor-market and macro indicators, built to estimate the U.S. U-3 unemployment rate for December 2025.

Group project for *Statistical Models for Data Science* (Analysis Challenge), University of Chicago.
**Group 8:** Devanshu Khadka, Mahima Goyal, Jiayi Hui, Sadaf Khan. Instructor: Prof. Ming-Long Lam.

## Approach

1. **Data.** Seven monthly series from FRED (UNRATE as target; JTSJOL, ICSA, CCSA, CPIAUCSL, FEDFUNDS, PAYEMS as inputs). Weekly claims were averaged to monthly. See [`data/README.md`](data/README.md).
2. **Feature engineering** (`notebooks/01`). Log transforms of claims, 1- and 3-month lags, month-over-month percent changes, and 3-month rolling averages. This gives 20 candidate predictors.
3. **Model selection** (`notebooks/02`). Forward selection using F-tests (entry threshold p = 0.05) on 294 monthly observations, ending with 10 predictors:
   `UNRATE_ma3, PAYEMS_pct, ICSA_lag1, CCSA_ma3, CCSA, ICSA_ma3, log_ICSA, PAYEMS, ICSA_lag3, PAYEMS_lag1`
4. **Final model.** OLS fit on the full history, with residual plots, a Q-Q plot, and variance inflation factors.

## Results (in-sample, from the notebook output)

| Metric | Value |
|---|---|
| R² | 0.9977 |
| Adjusted R² | 0.9976 |
| RMSE | 0.0945 |
| Residual variance | 0.0093 |

Model estimate from the final row of the dataset: **4.23%** (presented as the December 2025 estimate).

## Limitations and next steps

These matter when reading the numbers above:

- **No out-of-sample test.** All metrics are computed on the same data the model was fit and selected on. A time-based holdout or rolling-origin evaluation is the natural next step.
- **Target leakage.** `UNRATE_ma3` is a 3-month average that includes the current month's unemployment rate, i.e. the target. It is the first variable forward selection picks (coefficient ≈ 0.995), and it accounts for much of the very high R².
- **Same-month predictors.** Most predictors are measured in the same month as the target, so the model works as a nowcast. A true forecast needs inputs that are lagged so they are known before the target month is released.
- **Multicollinearity.** VIFs are extreme for `PAYEMS` and `PAYEMS_lag1` (both above 200,000), so individual coefficients are not reliable to interpret.
- **COVID-19 outliers** were kept in the data and are not modeled separately.

Suggested improvements: drop target-derived features or lag them, compare against a naive "last value" and an AR baseline, and evaluate with a rolling-origin backtest.

## Repository layout

```
notebooks/
  01_eda_and_feature_engineering.ipynb
  02_forward_selection_and_regression.ipynb
data/
  README.md                       # data sources and expected files
presentation/
  G8_U3_LaborPrediction_Dec2025.pptx
requirements.txt
```

## Running it

The notebooks were written in Google Colab.

1. Install dependencies: `pip install -r requirements.txt`
2. Put `monthly_panel.csv` and `unemployment_model_input.csv` where the notebooks read them (see [`data/README.md`](data/README.md)). Notebook 01 reads `monthly_panel.csv`; notebook 02 reads `/content/unemployment_model_input.csv`, so change that path if you run it locally.
3. Notebook 02 imports a `Regression` helper module (`import Regression`) provided in the course. It is not included in this repo; add `Regression.py` next to the notebook, or replace its calls with `statsmodels` OLS.
