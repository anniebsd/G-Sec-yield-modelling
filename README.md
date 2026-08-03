# G-Sec Yield Curve PCA

PCA-based factor decomposition and forecasting of the Indian G-Sec yield curve, built from raw CCIL trade data and macro-financial variables (policy repo rate, M3 money supply growth, US 10Y Treasury yield, USD/INR). The variable set follows Dua & Raje (2014), *"Determinants of Yields on Government Securities in India."*

## What this project does

1. Constructs a daily par-yield-equivalent G-Sec curve (1Y–30Y) from raw CCIL trade-by-trade data — filtering to central government fixed-coupon securities, bucketing trades by residual maturity, and computing face-value-weighted yields.
2. Assembles five supporting macro-financial variables (policy rate, money supply growth, a term spread, a foreign interest rate, and a CIP-implied forward premium proxy) from RBI DBIE and FRED.
3. Runs PCA on the standardized, weekly-resampled panel to decompose the system into a small number of interpretable factors.
4. *(In progress)* Forecasts the extracted factors forward and reconstructs forecasted yield curves, benchmarked against a random-walk model.

## Key findings so far

Three components explain **91.9%** of total variance across the eleven-variable system:

| Component | Variance Explained | Interpretation |
|---|---|---|
| **PC1** | 74.8% | A broad rate-and-currency regime factor — G-Sec yields, M3 growth, US 10Y, and USD/INR all move together, against a comparatively lower policy rate. |
| **PC2** | 12.9% | A classic slope factor — the short end (1Y) and the term spread move sharply opposite to the long end, which barely participates. |
| **PC3** | 4.2% | A domestic-policy-vs-global-rate factor, dominated by the repo rate and forward premium proxy moving against the US 10Y. |

A notable standalone finding: over the sample period, the repo rate was cut (5.50% → 5.25%) while G-Sec yields broadly rose — a real, counter-intuitive divergence between the policy rate and the wider curve, visible directly in the data before any PCA was run.

## Data sources

| Variable | Source | Frequency |
|---|---|---|
| G-Sec yields (1Y/2Y/5Y/10Y/15Y/30Y) | CCIL "G-Sec Historical Trades" | Daily (constructed) |
| Policy repo rate | RBI DBIE, Major Monetary Policy Rates | Event-driven, forward-filled |
| M3 money supply growth | RBI DBIE, Monetary Survey | Monthly, forward-filled |
| Foreign interest rate (US 10Y) | FRED, series DGS10 | Daily |
| Spot USD/INR | RBI DBIE, Exchange Rate of the Indian Rupee | Monthly, forward-filled |
| Term spread, forward premium proxy | Constructed from the above | — |

## Methodology notes and limitations

- Yields are standardized (zero mean, unit variance) before PCA, since raw variables span very different natural scales (single-digit yields vs. USD/INR in the 90s vs. double-digit M3 growth) — using an unstandardized covariance matrix would let scale, not economic significance, dominate the components.
- The forward premium proxy is constructed as `repo_rate − US 10Y`, both of which are separately included in the same matrix. This creates exact collinearity and forces one eigenvalue to be numerically zero — a direct, explainable consequence of that construction choice, not a modeling error.
- The dataset covers roughly one year (mid-2025 to mid-2026), which may not span a full monetary policy cycle.
- T-Bills were excluded from the yield curve construction to keep scope manageable; this removes visibility into the very short end (<1Y) of the curve.

Full reasoning and step-by-step analysis for each stage is in [`PROJECT_LOG.md`](PROJECT_LOG.md).

## Repository structure

- `gsec_pca_pipeline.ipynb` — main analysis notebook (data pipeline, PCA, forecasting)
- `PROJECT_LOG.md` — detailed week-by-week task log and tool list
- `LICENSE` — MIT

## Status

Week 2 of 4 complete (data pipeline + PCA decomposition). Factor forecasting and curve reconstruction in progress.

## Tools

Python (pandas, numpy, sklearn, statsmodels, matplotlib), Jupyter Notebook.
