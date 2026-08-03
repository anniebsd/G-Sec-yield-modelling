# G-Sec-yield-curve-PCA
PCA-based factor decomposition and forecasting of the Indian G-Sec yield curve, using RBI/CCIL trade data and macro-financial variables (repo rate, M3 growth, US 10Y, USD/INR).
### Week 1 — Data acquisition + cleaning + maturity bucketing


**Directly pulled from a source (raw data):**

- ☑ **G-Sec yields across maturities** (1Y, 2Y, 5Y, 10Y, 15Y, 30Y) — built from CCIL trade-by-trade data → `panel`
- ☑ **Policy repo rate** — RBI DBIE "Major Monetary Policy Rates" → `repo_daily`
- ☑ **Money supply (M3) growth** — RBI DBIE "Monetary Survey" → `m3_daily` (YoY % growth)
- ☑ **Foreign interest rate** — FRED, US 10Y Treasury yield (DGS10) → `us10y_daily`
- ☑ **Spot USD/INR** — RBI DBIE "Exchange Rate of the Indian Rupee..." (US Dollar End-month) → `usdinr_daily`

**Constructed/computed from the above (not separately downloaded):**

- ☑ **Interest rate spread** — `term_spread` = `panel['10Y']` − `panel['1Y']`
- ☑ **Forward premium proxy** (CIP-implied) — `forward_premium_proxy` = `repo_daily` − `us10y_daily`

**Final assembly:**

- ☑ **`final_matrix`** — all of the above joined into one dataframe on the daily `panel.index`, confirmed zero missing values

**Tools needed this week:**

- Browser (manual downloads from CCIL, RBI DBIE, FRED)
- Excel (only if useful for a first-pass eyeball of a messy file before loading into pandas — optional, not required)
- Python: `pandas` (loading, cleaning, merging, regex extraction, groupby aggregation), `numpy`, `matplotlib` (sanity-check plots)
- Jupyter Notebook as your workspace

**Done (data acquired):**

1. ✅ G-Sec yields across maturities — `panel`
2. ✅ Policy rate — `repo_daily`
3. ✅ Money supply growth (M3) — `m3_daily`
4. ✅ Interest rate spread — `term_spread`
5. ✅ Foreign interest rate — `us10y_daily`
6. ✅ Spot USD/INR — `usdinr_daily`

### Week 2 — PCA decomposition (Dua & Raje variable set)

**Tasks:**

- Assemble final input matrix: G-Sec yields across maturities + repo rate + M3 growth + interest rate spread + foreign rate + forward premium proxy, aligned to a common (likely weekly) frequency.
- Center the data, compute covariance matrix, run eigendecomposition.
- Extract PC1/PC2/PC3 (and check if a PC4 is warranted).
- Plot eigenvector loadings per variable, confirm/interpret what each component represents.
- Report variance explained per component.

**Tools needed this week:**

- Python: `pandas`, `numpy` (manual eigendecomposition for understanding), `sklearn.decomposition.PCA` (for convenience once you've done it manually once), `matplotlib`/`seaborn` for loading plots
- Jupyter Notebook

### Week 3 — Factor forecasting + walk-forward backtesting

**Tasks:**

- Set up random-walk benchmark for 1-day-ahead and 1-month-ahead horizons.
- Build AR(1) per component, then upgrade to VAR across components jointly.
- Implement proper walk-forward/rolling backtest (not a single in-sample fit).
- Compute RMSE for your model vs. random-walk benchmark, per component and per horizon.

**Tools needed this week:**

- Python: `statsmodels` (`tsa.vector_ar.var_model.VAR`, or `tsa.ar_model.AutoReg`), `pandas`, `numpy`, `matplotlib` (forecast vs. actual plots)
- Jupyter Notebook

### Week 4 — Curve reconstruction, error analysis, robustness, polish

**Tasks:**

- Reconstruct forecasted yields from forecasted PC scores (project back via eigenvectors, add back means).
- Plot forecasted vs. actual curves for sample future dates.
- Error analysis: RMSE per maturity point, performance around MPC meeting dates vs. ordinary days, high vs. low volatility regimes.
- Robustness checks: 2 vs. 3 components, different training window lengths.
- Clean notebook into a linear narrative; move reusable functions into a `utils.py`.
- Write the two deliverables: technical write-up (methodology, results, honest limitations) + 1-2 page non-technical summary connecting it to PD relevance (curve positioning, and — as an honest caveat — that this is a research/decision-support signal, not a live trading tool, given random-walk-like yield behavior and no event-awareness).
- UPLOAD TO GITHUB

**Tools needed this week:**

- Python: same stack as before, plus finalizing `.py` module structure
- Jupyter Notebook (final version) → optionally export to PDF/HTML for sharing
- Word or a markdown file for the non-technical summary (Excel not required at any point in this pipeline; only optional if you want a polished business-facing chart export at the very end)

---

**Summary of full tool stack across the month:**

- **Python/Jupyter**: your entire technical pipeline — `pandas`, `numpy`, `sklearn`, `statsmodels`, `matplotlib`/`seaborn`. Non-negotiable, used every week.
- **Browser**: manual data pulls from CCIL, RBI DBIE, RBI's monetary policy page, FRED — front-loaded mostly in Week 1, occasional revisits if you need to backfill something.
- **Excel**: optional, incidental only — occasional messy-data eyeballing or a final non-technical chart, never part of the core modeling.
- **Word/markdown**: for the final non-technical write-up in Week 4
