# G-Sec Yield Curve Modelling

Modelling and forecasting the Indian G-Sec yield curve using RBI/CCIL trade data and macro-financial variables (repo rate, M3 growth, US 10Y, SOFR, USD/INR, CPI, WPI). The variable set follows Dua & Raje (2014), *"Determinants of Yields on Government Securities in India."*

## What this project does
1. Constructs a daily par-yield-equivalent G-Sec curve (1Y–30Y) from raw CCIL trade-by-trade data — filtering to central government fixed-coupon securities, bucketing trades by residual maturity, and computing face-value-weighted yields.
2. Assembles supporting macro-financial variables (repo rate, M3 growth, US 10Y, SOFR, USD/INR, CPI, WPI) from RBI DBIE, FRED, and MoSPI.
3. Decomposes the standardized weekly panel via PCA, then tests whether the resulting factors are forecastable using ARIMA.
4. Builds a direct multi-variable regression of `india_10y` on the raw macro variables, using backward elimination and forward stepwise selection to identify significant predictors.
5. Validates the final model on a genuine out-of-sample holdout and on specific real 2026 dates the model never trained on.

## Methodology notes and limitations

The project went through several iterations, each motivated by a specific problem found in the previous step:

1. **PCA on the full panel.** Goal: decompose the yield curve and macro variables into a small number of interpretable factors. Ran PCA on the standardized weekly panel — but included `india_10y` itself in the input matrix. PC2 emerged as a strong single-factor predictor (correlated with the yield at -0.88, and later gave R² = 0.77 in a regression). Scrapped once I realized the target variable was inside the decomposition itself — PC2 wasn't predicting the yield, it was partly built from it.

2. **WPI data error.** While digging into what was driving PC2's strength, the WPI series turned out to be frozen at a single value for 60 consecutive weeks — a stale forward-fill masking a real data gap. Since WPI fed directly into PC2's loadings, this meant part of PC2's "signal" was actually a data artifact, not economics.

3. **ARIMA on the PCA factors.** Goal: test whether PC1/PC2, as standalone time series, were forecastable. Ran ADF tests, ACF/PACF, and fit several ARIMA orders. Result: AIC/BIC consistently favored the simplest possible order — (0,1,0), a pure random walk — over any AR/MA structure, and the resulting RMSE was equal to or worse than just forecasting the flat historical mean. Scrapped because there was no exploitable autocorrelation to forecast, and the target was still partly corrupted by the WPI issue above — so even a working ARIMA would have been fitting noise.

4. **Trimming and rebuilding.** Cut the dataset to July 2021–Dec 2025, where the data was verified clean, and reran the whole variable-selection process from scratch — backward elimination and a forward "ladder" method — using the raw macro variables directly instead of PCA factors.

5. **CPI/WPI base-year error.** The trimmed data still looked wrong. Traced it to CPI and WPI each switching base years partway through the series (2012-base vs. 2024-base) without reconciliation — CPI was frozen at 1.33% for 30 straight weeks as a result. Fixed by splicing the two base-year series at the correct cutover point.

6. **Final regression.** With clean data, reran backward elimination and the ladder method independently — both converged on the same variable set and coefficients, confirming the selection wasn't a fitting artifact. Validated with walk-forward backtesting, then forecast `india_10y` for three real 2026 dates the model never trained on, feeding in each date's actual macro values and comparing the prediction against the true yield.
   *(Next step: extend this to unconditional forecasting — predicting future yields using only information available today, rather than that period's realized macro values.)*

**Other technical notes:**
- Variables are standardized (zero mean, unit variance) before PCA, since raw variables span very different natural scales (single-digit yields vs. USD/INR in the 90s vs. double-digit growth rates) — an unstandardized covariance matrix would let scale, not economic significance, dominate the components.
- The current final model is a **conditional regression**: it uses each week's actual macro values to explain/predict that week's yield. It has not yet been extended to forecast using only prior information.
- T-Bills were excluded from the yield curve construction to keep scope manageable; this removes visibility into the very short end (<1Y) of the curve.

Full reasoning and step-by-step analysis for each stage is in [`PROJECT_LOG.md`](https://github.com/anniebsd/G-Sec-yield-modelling/blob/main/PROJECT_LOG.md).

## Repository structure
- `final_model.ipynb` — final, clean analysis notebook (data pipeline, backward elimination + ladder selection, holdout and real-date validation)
- `final_draft.ipynb` — full working notebook showing the iterative process (PCA, PCA leakage fix, ARIMA testing, data error discovery and fixes, trimming, final regression)
- `PROJECT_LOG.md` — detailed week-by-week task log and tool list
- `LICENSE` — MIT

## Status
Core pipeline complete: data cleaning, PCA exploration, ARIMA testing (scrapped), and final multi-variable regression validated against a genuine 2026 holdout. Still to do: extend the model to unconditional forecasting — predicting future yields from only information available at the time of prediction, rather than that period's realized macro values.

## Tools
Python (pandas, numpy, scipy, sklearn, statsmodels, matplotlib, seaborn), Jupyter Notebook.
