> **Key takeaways:** Built a regression model of the Indian 10Y G-Sec yield using ~4.5 years of RBI/CCIL trade data and macro variables (repo rate, M3, US 10Y, SOFR, USD/INR, CPI, WPI). Along the way, caught and fixed two real data errors that would have silently corrupted the results — a frozen/stale WPI series and a CPI/WPI base-year splice issue — plus a PCA leakage bug where the target variable had been included in its own factor decomposition. After fixing these and rebuilding the pipeline, the final 5-variable regression achieved an out-of-sample RMSE of **0.1248** on a genuine Jan–Jul 2026 holdout it never trained on.


# G-Sec Yield Curve Modelling

Modelling and forecasting the Indian G-Sec yield curve using RBI/CCIL trade data and macro-financial variables (repo rate, M3 growth, US 10Y, SOFR, USD/INR, CPI, WPI). The variable set follows Dua & Raje (2014), *"Determinants of Yields on Government Securities in India."*

G-Sec yields don't move in lockstep with the policy rate — over the sample period, the repo rate was cut (5.50% → 5.25%) while G-Sec yields broadly *rose*, a real divergence driven by fiscal and liquidity pressures rather than monetary policy alone. Understanding which macro variables actually move the curve, and how reliably, is directly relevant to rates desks, treasury functions, and macro/fixed-income research.

## What this project does
1. Constructs a daily par-yield-equivalent G-Sec curve (1Y–30Y) from raw CCIL trade-by-trade data — filtering to central government fixed-coupon securities, bucketing trades by residual maturity, and computing face-value-weighted yields.
2. Assembles supporting macro-financial variables (repo rate, M3 growth, US 10Y, SOFR, USD/INR, CPI, WPI) from RBI DBIE, FRED, and MoSPI.
3. Decomposes the standardized weekly panel via PCA, then tests whether the resulting factors are forecastable using ARIMA.
4. Builds a direct multi-variable regression of `india_10y` on the raw macro variables, using backward elimination and forward stepwise selection to identify significant predictors.
5. Validates the final model on a genuine out-of-sample holdout and on specific real 2026 dates the model never trained on.

## Findings so far

The final model, fit on data through Dec 2025, is: india_10y = 5.4344 − 0.0467·m3_growth + 0.0706·us_10y + 0.2289·sofr + 0.1255·cpi + 0.0524·wpi
Tested on three real 2026 dates the model never trained on, using each date's actual realized macro values:

| Date | Predicted `india_10y` | Actual `india_10y` | Error | Error % |
|---|---|---|---|---|
| 2026-02-03 | 6.5104% | 6.721% | -0.2106 | -3.13% |
| 2026-03-06 | 6.4848% | 6.688% | -0.2032 | -3.04% |
| 2026-04-01 | 6.8534% | 6.960% | -0.1066 | -1.53% |

The model consistently **underpredicts** the actual yield across all three dates, though the gap narrows over time (-3.13% → -3.04% → -1.53%). This is a systematic bias worth investigating further — a missing variable, a level shift, or possibly a sign/scale issue in one of the coefficients — rather than something that will average out with more data points.

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

## Known limitations & next steps

- **Conditional, not unconditional, forecasting.** The final model predicts `india_10y` using that same week's *actual* macro values — it hasn't yet been extended to forecast using only information available before the prediction date.
- **Stationarity/cointegration not yet tested.** The regression is run on levels, not differences. Since yields and rates are typically non-stationary, this raises a risk of spurious regression that hasn't been formally ruled out yet — a stationarity/cointegration check is the next planned addition.
- **Small out-of-sample evaluation set.** The three hand-picked 2026 dates in the Findings section are illustrative; the full holdout evaluation (Jan–Jul 2026, all weeks) gives RMSE = 0.1248 and is the more representative number.
- **Next step:** extend to unconditional forecasting, add stationarity/cointegration diagnostics, and evaluate across the full holdout period rather than a handful of example dates.

## Repository structure
- `final_model.ipynb` — final, clean analysis notebook (data pipeline, backward elimination + ladder selection, holdout and real-date validation)
- `final_draft.ipynb` — full working notebook showing the iterative process (PCA, PCA leakage fix, ARIMA testing, data error discovery and fixes, trimming, final regression)
- `PROJECT_LOG.md` — detailed week-by-week task log and tool list
- `LICENSE` — MIT

## Status
Core pipeline complete: data cleaning, PCA exploration, ARIMA testing (scrapped), and final multi-variable regression validated against a genuine 2026 holdout. Still to do: extend the model to unconditional forecasting — predicting future yields from only information available at the time of prediction, rather than that period's realized macro values.

## Tools
Python (pandas, numpy, scipy, sklearn, statsmodels, matplotlib, seaborn), Jupyter Notebook.
