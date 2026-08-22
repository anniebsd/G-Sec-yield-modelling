# GSEC Model Report

## Contents

1. [Executive Summary](#1-executive-summary)
2. [Introduction](#2-introduction)
3. [Literature and Background](#3-literature-and-background)
4. [Data Sources](#4-data-sources)
5. [Methodology](#5-methodology)
6. [Results](#6-results)
7. [Limitations](#7-limitations)
8. [Conclusion](#8-conclusion)

---

## 1. Executive Summary

The India 10-Year G-Sec yield is the benchmark long-term interest rate for India's fixed-income market, anchoring everything from corporate bond spreads to bank lending rates. Because it responds to a wide range of forces — global rate movements, inflation, money supply growth, and currency conditions — rather than any single driver like the policy repo rate, this report models it using several years of RBI/CCIL trade data and macro-financial variables, testing three methodological approaches before arriving at a validated final specification.

An initial PCA decomposition of the standardized macro panel found a small number of components capturing the large majority of total variance in the system. After correcting for a target-leakage issue (excluding the yield itself from the decomposition), one component retained a strong standalone correlation with the yield. However, formal stationarity and structure testing showed the extracted components were statistically indistinguishable from a random walk, with no forecast improvement over simply assuming no change. This indicated no exploitable time-series structure to forecast from the factors alone, and the approach was correctly abandoned on that evidence rather than forced.

Pivoting to a direct multi-variable regression surfaced a critical data integrity issue: CPI was found frozen at a single value for an extended, implausible stretch of consecutive weeks, traced to an unreconciled base-year discontinuity in the source series, and corrected using an updated, methodologically consistent dataset. With the corrected data trimmed to a verified-clean window, backward elimination identified a small set of statistically significant macro drivers — money supply growth, the US 10-Year yield, SOFR, CPI, and WPI — each significant at conventional confidence levels and having magnitude consistent with expected transmission channels, such as global rate spillover from US monetary conditions into Indian yields.

The model was validated genuinely out-of-sample: macro inputs were independently forecast forward using only historical information, run through the fixed regression, and checked against real yield data. Decomposing the result further isolated the residual forecast gap as a structural limitation of the five-variable specification itself, rather than of the forecasting method — the model's own accuracy using real inputs was no better than its forward forecast, pointing to fiscal, auction-supply, or sentiment effects outside its current scope as the most promising direction for future extension.

---

## 2. Introduction

### (a) Indian G-Sec Yields and Macroeconomic Variables

**(i)** The Government of India Security (G-Sec) market is the backbone of India's fixed-income system — the sovereign yield curve against which nearly every other domestic interest rate is priced, from corporate bond spreads to bank lending rates to the discount rates used in equity valuation. Among the various points on this curve, the 10-year G-Sec yield holds particular significance: it is the most widely quoted, most liquid, and most closely watched maturity, serving as India's de facto benchmark long-term interest rate.

**(ii)** The 10Y yield matters to a wide range of market participants for different reasons. For the Reserve Bank of India, it is both an indicator of market expectations around inflation and growth, and — through its interaction with the policy repo rate — a signal of how effectively monetary policy is transmitted through the financial system. For banks and institutional investors, it anchors duration and interest-rate-risk decisions across trillions of rupees in fixed-income portfolios. For the central government, it directly determines the cost of borrowing on new debt issuance, with even small movements translating into meaningful shifts in the fiscal deficit's interest burden. For foreign investors, it is a key input into relative-value decisions between Indian and global sovereign debt, particularly against benchmarks like the US 10-Year Treasury.

**(iii)** Despite this importance, the 10Y yield does not move in a simple, mechanical relationship with any single variable, say repo rates; instead it suggests that yields are shaped by a wider set of forces — global rate movements, inflation dynamics, money supply growth, and currency conditions among them — that a policy-rate-only view would miss entirely.

### (b) Objective

**(i)** The objective of this report is to build a model of the India 10Y G-Sec yield that can identify which macro-financial variables genuinely explain yield movements, quantifies the direction and magnitude of their influence, and can be used to generate forward-looking forecasts grounded in real economic reasoning rather than black-box extrapolation.

**(ii)** Interpretability is treated primarily in this model. A model that predicts yields accurately but cannot explain why is of limited use to the audiences described above — a rates desk, a treasury function, or a macro research team needs to understand which levers are moving the curve, not just what the curve is expected to do next. For this reason, the analysis deliberately favours transparent, coefficient-based methods over more opaque alternatives, and treats every step of model-building as an opportunity to test and validate an economic hypothesis, not merely to optimize a fit statistic.

**(iii)** Economic relevance is the second guiding principle. Every variable considered in this analysis — the policy repo rate, US 10Y Treasury yield, SOFR, CPI and WPI inflation, and M3 money supply growth — was selected because it has a plausible, well-established transmission channel into sovereign bond yields, whether through monetary policy stance, inflation expectations, global capital flows, or liquidity conditions. The goal is not simply to find whatever combination of variables maximizes R², but to build a model whose structure an investment analyst or policymaker would recognize and trust. This report aims to present the reasoning behind what was kept, what was abandoned, and why.

---

## 3. Literature and Background

### (a) The Established Variable Set

The empirical literature on emerging-market and Indian government bond yields converges on a fairly stable set of explanatory variables, despite differences in sample period, estimation method, and market studies. The foundational reference for this analysis is Dua and Raje's (2014) study of Indian government security yields, which used weekly data from April 2001 through June 2012 across treasury bills and government securities of 1, 5, and 10-year maturities. Their central finding, in turn, was that a long-run relationship exists between each of these yields and six variables: the policy rate, the rate of growth of money supply, inflation, the interest rate spread, a foreign interest rate, and a forward premium. Notably, they found that the relative importance of these determinants shifts across the maturity spectrum — with the policy rate and money supply growth exerting more explanatory power over shorter-term yields than over longer-term ones, a finding directly relevant to this report's focus on the 10-year tenor, where domestic policy variables might reasonably be expected to matter somewhat less than global and inflation-linked factors.

This variable set is not unique to the Indian context — the same broad categories recur throughout the wider literature on sovereign bond yield determinants, whether the market studied is emerging or developed. Four categories account for most of the explanatory power across studies:

- **Monetary policy stance** — typically proxied by the central bank's policy rate — captures the near-term interest rate environment set by the domestic monetary authority, and is the most direct channel through which policy affects the front end of the yield curve.
- **Inflation and inflation expectations** — usually measured via realized CPI and/or WPI — matter because bond yields are, in essence, compensation for expected future purchasing power loss; a rise in expected inflation should mechanically demand higher nominal yields to preserve real returns.
- **Money supply / liquidity conditions** — commonly measured via M3 growth — reflect the broader liquidity environment in the domestic banking system, which affects both the supply of loanable funds and inflation expectations over a medium-term horizon.
- **Global interest rates and capital flow spillovers** — this is the category most emphasized in the EM-specific literature beyond Dua and Raje's original domestic-focused framework. A substantial body of subsequent work examines how global monetary conditions, and US monetary policy in particular, transmit into local-currency emerging-market bond yields, given the depth of foreign participation in these markets and their sensitivity to global risk appetite and capital flows.

### (b) Weaker Counterparts

Not every macro variable with a plausible economic story survives empirical scrutiny in this research. Exchange rate levels (such as USD/INR) are frequently included as a control in EM bond yield studies, on the theory that currency depreciation raises the risk premium foreign investors demand — but their explanatory power relative to the core variable set above is often modest once inflation and global rates are already accounted for, since much of the currency channel's effect is thought to work indirectly through those same variables rather than independently. Domestic equity market volatility and risk sentiment measures, similarly, are sometimes included as controls but rarely emerge as first-order drivers in the way that policy rates, inflation, and global rates consistently do.

This report's variable selection, and its use of systematic elimination procedures rather than judgment alone to arrive at a final specification, is designed to let the data confirm or challenge this literature's priors directly — rather than assuming, a priori, that every commonly-cited variable will prove significant in this specific sample.

---

## 4. Data Sources

### (a) Overview

The final model draws on daily and monthly data spanning roughly 4.5 years, sourced from a mix of official Indian government/regulatory publications and standard international benchmarks. Every series was independently sourced, cleaned, and aligned to a common daily date index before being resampled to a weekly (Friday) frequency for modelling — the frequency at which the dependent variable, the India 10Y G-Sec yield, is itself constructed from raw trade data.

| Variable | Description | Source | Native Frequency | Alignment Method |
|---|---|---|---|---|
| `india_10y` | India 10-Year G-Sec par yield | CCIL trade-by-trade data, face-value-weighted, bucketed to nearest residual maturity | Daily (constructed) | Native |
| `repo_rate` | RBI policy repo rate | RBI DBIE, "Major Monetary Policy Rates" | Event-driven (changes at MPC decisions) | Forward-filled to daily |
| `m3_growth` | M3 money supply, YoY % growth | RBI DBIE, "Monetary Survey" | Monthly | Forward-filled to daily |
| `us_10y` | US 10-Year Treasury yield | FRED, series DGS10 | Daily | Native |
| `sofr` | Secured Overnight Financing Rate | FRED, series SOFR | Daily | Native |
| `cpi` | India CPI (Combined), YoY % inflation | Official CPI press release data (MoSPI) | Monthly | Forward-filled to daily |
| `wpi` | India WPI, All Commodities, YoY % inflation | Official WPI press release data (Ministry of Commerce & Industry) | Monthly | Forward-filled to daily |
| `usdinr` | Spot USD/INR exchange rate | RBI DBIE, "Exchange Rate of the Indian Rupee" | Monthly (end-of-month) | Forward-filled to daily |

All series were joined on `india_10y`'s daily trading-day index, then resampled to weekly (Friday) frequency for the modelling stages that follow.

### (b) Base-Year in CPI and WPI

Two of the seven macro variables — CPI and WPI — required special handling beyond routine cleaning, due to a structural issue common to official Indian price indices: periodic base-year revisions.

Both CPI and WPI are published as index levels relative to a fixed base period, which the compiling ministries revise periodically to keep the underlying consumption/production basket representative of the current economy. During the sample period covered by this analysis, both series underwent a base-year transition — CPI to a 2024 base, and WPI to a 2022–23 base — from their respective legacy base periods. Critically, a base-year revision is not merely a re-scaling exercise: the underlying item basket, weights, and in some cases methodology are also revised, meaning the pre- and post-revision series are not simply two segments of the same continuous number line. Directly concatenating or forward-filling across the transition point, without accounting for this, risks producing a series that is internally inconsistent — appearing to show no change (or a mechanically incorrect jump) at the cutover, when the reality is a methodology break, not an economic one.

This is precisely the failure mode that surfaced during model-building (detailed further in methodology): the initial CPI series, built from a single legacy-base source, was found frozen at a single value for over 30 consecutive weeks. The resolution was to source a newer, correctly-updated dataset that reflected each index on its current base year (2024 for CPI, 2022–23 for WPI) throughout the full sample period, ensuring the calculation methodology was internally consistent across the entire series.

### (c) Other Data Quality Notes

Beyond the base-year issue, several series required routine but non-trivial cleaning before use:

- G-Sec yields were taken from CCIL trade-by-trade data and were filtered to central government fixed-coupon securities, bucketed by residual maturity to the nearest standard tenor (1Y, 2Y, 5Y, 10Y, 15Y, 30Y), and aggregated into a face-value-weighted daily yield per bucket, since no single continuously-traded bond exists at exactly 10 years' maturity at all times. They were consolidated in a single Excel workbook alongside some of the other variables.
- Repo rate and USD/INR are naturally sparse, event-driven or monthly series, and were forward-filled to a daily frequency under the standard assumption that a rate or rate-equivalent remains constant between official updates.
- T-Bills were excluded from the yield curve construction entirely, to keep scope manageable; this means the analysis has no visibility into the very short end (under 1 year) of the curve.

---

## 5. Methodology

### (a) Principal Component Analysis

The analysis began from a factor-based hypothesis: rather than model the India 10Y yield directly against seven individual macro variables, the first approach tested whether a small number of extracted factors could summarize the system more efficiently, following a standard dimensionality-reduction logic common in yield curve and macro-factor literature.

The full panel — the India 10Y yield alongside repo rate, M3 growth, US 10Y, SOFR, India VIX, USD/INR, CPI, and WPI — was resampled to weekly frequency and standardized (zero mean, unit variance) prior to decomposition. Standardization is necessary here because the raw variables span very different natural scales (single-digit yields, USD/INR in the 90s, double-digit growth rates), and an unstandardized covariance matrix would let scale, rather than economic significance, dominate the resulting components.

An initial eigendecomposition of the full nine-variable standardized panel found the first three components explaining a combined 91.9% of total variance, with PC1 alone accounting for 74.8%. However, this first pass included `india_10y` itself as one of the input variables to the decomposition — a design choice that, on reflection, introduces a subtle but serious methodological problem: any component correlated with the yield in this setup is correlated partly because the yield helped construct it.

Correcting for this, the PCA was re-run on the eight remaining macro variables with `india_10y` excluded entirely from the input matrix. This re-run found a materially different variance structure: PC1 explained 52.8%, PC2 21.4%, and PC3 11.9% of total variance among the eight remaining variables. Despite the yield's removal from the decomposition, PC2 retained a strong standalone correlation with the yield of −0.879, and a simple univariate regression of `india_10y` on PC2 alone produced an R² of 0.772 — evidence that PC2's relationship with the yield was a genuine feature of the underlying macro data, not an artifact of leakage. Further examining PC2's loadings identified it as being driven predominantly by the interest-rate-and-currency cluster of variables (US 10Y, SOFR, USD/INR moving together against the domestic policy rate) — economically interpretable as a global-rate-and-currency regime factor, broadly consistent with the "foreign interest rate" channel emphasized in Dua and Raje's framework.

### (b) Testing Forecastability Through ARIMA

Having confirmed PC1 and PC2 carried genuine explanatory power over the yield in-sample, the next question was whether these factors were themselves forecastable — that is, whether their own future values could be predicted from their own past behavior, which would allow a two-stage forecast (forecast the factor, then map the factor to a yield forecast).

This was tested via the standard Box-Jenkins ARIMA framework. Augmented Dickey-Fuller tests on both PC1 and PC2 indicated non-stationarity in levels, requiring first-differencing (d=1). ACF and PACF plots of the differenced series showed no significant autocorrelation structure beyond lag zero — a first indication that little exploitable structure existed. This was confirmed formally: fitting ARIMA models of orders (0,1,0), (1,1,0), (0,1,1), and (1,1,1) to both factors, the simplest possible specification — (0,1,0), a pure random walk with no autoregressive or moving-average terms — consistently minimized both AIC and BIC for both PC1 and PC2, indicating that adding AR or MA structure did not improve the model enough to justify the additional parameters.

The practical implication was tested directly via out-of-sample forecasting on a held-out test split (85/15 train/test): the (0,1,0) ARIMA forecast for PC1 produced an RMSE of 0.2506 against a test-period standard deviation of 0.2443 — meaning the model's forecast error was statistically indistinguishable from simply predicting no change at all. A rolling-window backtest (150-week training window, 4-week forecast horizon, 27 windows tested) confirmed this was not an artifact of a single train/test split: mean RMSE across all windows was 0.1836 for PC1 and 0.2549 for PC2, with substantial variance across windows (std. 0.1035 and 0.1436 respectively), indicating unstable and unreliable forecast performance regardless of the specific window chosen.

Hence, the extracted factors, despite explaining the yield well in-sample, showed no exploitable time-series structure of their own. Forecasting the yield via factor-forecasting was abandoned on this evidence, and the analysis pivoted toward a direct regression approach using the raw macro variables instead of PCA-derived factors.

### (c) Data Integrity Investigation

During the pivot to a direct regression approach, the source data itself came under closer scrutiny. In brief: the CPI series was discovered frozen at a single value for over 30 consecutive weeks, traced to an error in taking the base year data sets for CPI and WPI; they were found to be indexed on an older base year. Because a base-year revision typically involves a revised item basket and weighting methodology rather than a simple rescaling, the pre- and post-revision series could not be reliably spliced or forward-filled across the transition point without risking a misrepresentation of inflation.

This was resolved by sourcing updated datasets for both CPI and WPI, each reflecting the series consistently on its current base year across the full sample period. Some portion of PC2's apparent explanatory strength previously may itself have been partly traced to this flaw rather than pure economic signal — a caveat worth carrying forward when interpreting the PCA findings, even though PC2's correlation with the yield remained strong on the corrected regression-based approach that followed.

### (d) Direct Regression on Macro Variables

With corrected data in hand, the dataset was trimmed to a verified-clean window (from July 2021 to December 2025) and a direct multiple linear regression of `india_10y` on the seven raw macro variables (repo rate, M3 growth, US 10Y, SOFR, USD/INR, CPI, WPI) was constructed, now that the factor-based approach had been set aside.

Variable selection was performed via backward elimination: starting from the full seven-variable specification, the least statistically significant variable (highest p-value) was iteratively removed and the model refit, continuing until every remaining variable was significant at the 5% level. To guard against the possibility that this sequential, path-dependent procedure had converged on one specification-dependent result, a second, independent selection method — a forward "ladder" procedure, adding variables one at a time in order of individual univariate explanatory power, retaining each addition only if it remained statistically justified — was run separately on the same data.

Both methods converged on an identical final variable set and, notably, identical coefficients to six decimal places — strong evidence that the selected specification reflects a genuine, stable relationship in the data rather than an artifact of one particular selection path, with R²=0.738.

### (e) Model Validation Approach

Before generating forward-looking macro inputs, the model was tested on its real-world forecasting value by evaluating observed yield data for three dates across February, March, and April of 2026. Using the coefficients from the trained model, predictions were generated by feeding in each date's actual, known macro inputs and comparing against the true realized yield — the best result came on April 1st, with an error of −1.53%. Forecasting was then extended to a genuine forward test: independently forecasting the macro variables themselves across a three-month window into 2026.

### (f) Macro Input Projection

The holdout validation described above tests the model's fit against dates where the true macro inputs were already known and observable — a genuine out-of-sample test of the fitted relationship, but not yet a test of end-to-end forecasting, since it does not require predicting what the macro variables themselves will be. A true forward forecast — one usable before the fact, with no future information available — requires an additional step: independently forecasting each macro driver forward, then feeding those forecasted values through the fixed regression to produce a forecasted yield.

This makes the resulting forecast explicitly conditional: it is only as accurate as the projected macro inputs it depends on. Each macro driver was forecast using simple linear trend extrapolation, fit only on data prior to the forecast origin date, with the specific method tailored to each variable's underlying update frequency:

- **Continuously-updating variables** (US 10Y, SOFR, M3 growth) were extrapolated via ordinary least squares trend-fitting on a trailing window of 52 weekly observations, projected forward linearly.
- **Monthly-sourced variables** (CPI and WPI) are published monthly, but the model runs on a weekly grid, so each month's reading is repeated across all its weeks. Fitting a trend directly on this repeated weekly series would let months with more weeks unfairly dominate the trend estimate. To avoid this, the trend was instead fitted on the true monthly values only (using a trailing 12-week/3-month lookback), projected forward month by month, and then each forecasted monthly value was spread back across its corresponding weeks — preserving the step-like pattern these series actually follow.

An early version of this approach, using a long (52-week) lookback window uniformly across all variables, produced economically implausible results for CPI and WPI specifically — including negative forecasted inflation — because a multi-year linear trend, when extrapolated from series that do not move in straight lines indefinitely, can overshoot badly. Taking 12 weeks for these variables reflects the fact that YoY inflation can shift materially within a quarter in a way that a year-long trend window would smooth over or misrepresent.

### (g) Forecast Evaluation

The forecasting pipeline was validated by generating a genuine forward forecast for a full quarter, using only data available before that quarter began, and subsequently comparing it against two benchmarks: a hindsight version of the model fed the actual, later-released macro values for the same period, and the real observed yield itself.

This produced a natural three-way decomposition, used throughout the results to separate distinct sources of forecast error:

1. **The forecast itself** — the model's predicted yield, using trend-extrapolated macro inputs.
2. **A hindsight benchmark** — the model's predicted yield using the real, subsequently-observed macro inputs for the same period, isolating what the regression alone would have produced with perfect knowledge of the drivers.
3. **The real observed yield** — sourced independently after the forecast period concluded, providing genuine ground truth.

Comparing (1) against (3) gives total forecast error. Comparing (2) against (3) tells us how far off the model is even when it isn't guessing anything — i.e., this is the error baked into the model itself, regardless of forecasting skill. Comparing (1) against (2) isolates how much the trend-extrapolation method itself contributed to, or mitigated, the total error. This decomposition is what allows the results section to state precisely why the forecast was as accurate (or inaccurate) as it was.

---

## 6. Results

**Summary:** The final specification — five macro variables (M3 growth, US 10Y yield, SOFR, CPI, and WPI) selected via backward elimination and cross-confirmed by an independent forward stepwise procedure — explains 73.8% of the variation in the India 10Y yield (R²=0.738, Adj. R²=0.732), with the model jointly significant at p<0.001 and every individual coefficient significant at the 5% level. The largest economic effects come from SOFR (coefficient 0.229, meaning a 100bp move associates with a ~23bp move in the Indian yield) and CPI (0.126, a ~13bp move per 100bp of inflation), consistent with global rate spillover and domestic inflation-expectations channels respectively; US 10Y and WPI carry smaller positive coefficients (0.071 and 0.052), while M3 growth is the only variable with a negative sign (−0.047), consistent with a liquidity-driven effect once inflation is separately controlled for.

Out-of-sample testing confirmed the model generalizes beyond its training window. On three individually verified 2026 dates, predictions came within 1.5–3.1% of the realized yield, with the error narrowing over time as the model's inputs increasingly reflected a real acceleration in WPI. A fuller test — forecasting all five inputs forward across the first quarter of 2026 using only pre-2026 data, then checking against real subsequently-released yields — produced a mean absolute error of 17 basis points (MAPE 2.51%), tightening to single-digit basis-point errors by the final weeks of the quarter. Notably, this forecast performed marginally better (RMSE 0.180) than a hindsight version of the model fed the true, contemporaneous macro values (RMSE 0.197) — indicating the residual gap between prediction and reality stems primarily from the model's own specification, not from any imprecision in forecasting its inputs.

Diagnostic testing surfaced three notable results: residuals showed no significant departure from normality (Jarque-Bera p=0.169), supporting the validity of the reported significance tests; residual variance was not constant (Breusch-Pagan p≈0.000), indicating standard errors are somewhat approximate; and SOFR and WPI exhibited high multicollinearity (VIF 20.6 and 15.4 respectively) — tested directly by refitting without SOFR, which reduced R² to 0.675 and degraded out-of-sample MAPE to 2.66%, confirming SOFR's inclusion is empirically justified despite its overlap with related rate variables.

### Tables and Graphs

**(1) Final daily data matrix: shape and null check**

Output of the fully merged daily panel (`final_matrix`) — 1,121 daily rows × 9 columns (`sofr`, `usdinr`, `us_10y`, `india_10y`, `cpi`, `wpi`, `repo_rate`, `m3_growth`, plus a source flag), zero nulls across the board, with the tail showing late-December 2025. This confirms four of eight variables (`repo_rate`, `m3_growth`, `cpi`, `wpi`) originate as monthly or event-driven series and had to be forward-filled to daily frequency to align with the daily market series (`sofr`, `usdinr`, `us_10y`, `india_10y`). The zero-null result confirms the alignment/ffill logic worked across the full 2021–2025 window with no gaps.

**(2) Final OLS model (backward elimination)**

The regression of `india_10y` on all seven candidate macro variables, with `usdinr` (p=0.8039) and `repo_rate` (p=0.0963) sequentially dropped for exceeding the 5% significance threshold. Final model: R² = 0.738, Adj. R² = 0.732, F = 128.4 (p ≈ 3×10⁻⁶⁴), five significant predictors.

The coefficients for each of the five variables tell an economically coherent story. A negative coefficient for M3 growth shows the textbook Keynesian liquidity effect: an expansion in money supply increases the stock of liquidity in the banking system, and in the short-to-medium run that excess liquidity is partly absorbed into government securities, bidding up bond prices (pushing yields down) faster than inflation expectations catch up. A rise in SOFR raises the cost of borrowing dollars to fund EM carry trades and tends to trigger capital outflows from markets like India's — investors need a higher India yield to keep being compensated for that outflow/currency risk. The size of this coefficient (roughly 3× the `us_10y` coefficient) suggests the funding-cost/capital-flow channel matters more for India's 10Y than the pure term-premium channel captured by `us_10y`.

This also tells us that once `us_10y`, `sofr`, and `cpi` are already in the model, USD/INR carries no additional information. This is consistent with uncovered interest parity intuition — INR's exchange-rate movements are themselves largely a function of the same rate differentials and inflation dynamics already in the model, so including it separately is close to redundant.

**(3) Stepwise ladder selection, univariate ranking + step log**

At the raw bivariate level, faster money growth is probably positively correlated with inflation (CPI/WPI) — periods of loose money are periods of rising prices — and that positive inflation-driven relationship with yields is masking `m3_growth`'s true, independent liquidity effect. Only once CPI and WPI are held constant does the "pure" liquidity channel (more money chasing bonds → lower yields) emerge from underneath the offsetting inflation channel. Also worth noting: `repo_rate`'s univariate R² (0.195) and CPI's (0.361) dwarf `m3_growth`'s — CPI alone explains over a third of the variance in the 10Y yield, underscoring inflation as the dominant single driver of the Indian curve over this sample, with the rate/liquidity variables adding incremental explanatory power on top.

**(4) In-sample fit chart (July 2021–Dec 2025, weekly)**

Actual vs. fitted `india_10y` over the full training sample. In-sample RMSE = 0.2005 (i.e., ~20 basis points average error) against a series that ranges roughly 6.0%–7.6%.

**(5) Fitted formula and coefficients**

```
india_10y = 5.4344 − 0.0467·m3_growth + 0.0706·us_10y + 0.2289·sofr + 0.1255·cpi + 0.0524·wpi
```

**(6), (7), (8) Three point forecasts on real, later-verified data**

These are the three checks done against actual macro releases using the fixed model coefficients. All three real-world forecasts undershoot the actual yield, by roughly 11–21bps (1.5%–3.1% in relative terms). This isn't noise scattered around zero — it's a one-directional bias, and that's economically more informative than a single error number would be. A model that systematically underpredicts the level of the 10Y yield in early 2026 is missing something that's been pushing yields up beyond what `m3_growth`/`us_10y`/`sofr`/`cpi`/`wpi` alone predict — other variables from the literature, or other systematic risks. Another possibility is simply that five macro levels can't fully substitute for the yield's own momentum.

**(9) Trend-extrapolated forecast inputs (Jan–Mar 2026, Q1)**

The output of `forecast_linear_trend` / `forecast_monthly_trend` — each of the five model predictors extrapolated forward 12 weeks using a simple linear trend fit on the trailing 52 weeks (market variables) or trailing 3 months (CPI/WPI), without using any real 2026 data. It assumes whatever direction each variable was moving in through late 2025 continues linearly. That's a strong assumption for variables like SOFR and US 10Y, which are policy-driven and can plateau or reverse discontinuously, and it's visibly a weak assumption for CPI/WPI here: the extrapolated CPI rises smoothly from 1.69% to 2.82% and WPI from 0.98% to 2.45% across the twelve weeks.

**(10) Forecast chart with 95% prediction interval**

The complete 2021–2025 actual series plus the trend-extrapolated Q1 2026 forecast path, with a shaded 95% prediction interval band.

**(11) Zoomed version**

**(12) Real (non-extrapolated) Q1 2026 macro inputs**

Verified macro values for the five model variables across the same Jan–Mar 2026 weeks — i.e., what would be fed to the model in hindsight, once the real releases were known, instead of the linear trend extrapolation.

**(13) Forecast vs. model-on-real-inputs comparison table**

`forecast_india_10y` (model fed the trend-extrapolated inputs) vs. `actual_india_10y_from_real_inputs` (the same fixed model, but fed the real inputs) — isolating how much of the forecast error comes from bad inputs vs. a bad model. The two columns track closely (errors of roughly 1–3.6 basis points on the yield scale across the twelve weeks).

**(14) Graph**

**(15) RMSE decomposition**

The complete decomposition — forecast (trend inputs), hindsight (real inputs, same fixed model), and actual realized `india_10y` — with `forecast_error` and `model_hindsight_error` computed against the real yield. The forecast using extrapolated, imperfect inputs (RMSE 0.180) did slightly better than the model using the real, correct inputs in hindsight (RMSE 0.197). In other words, input-extrapolation error was not the main source of forecast error — the model's own residual bias (the systematic undershoot seen earlier) was. This flips the natural assumption (that a forecasting model's errors mostly come from not knowing the future inputs) and is a genuinely strong, citable finding: even with a crystal ball on the five macro inputs, this model would still have underpredicted the 10Y yield by about the same amount. That points to a limitation squarely at the model specification itself (missing lag structure, missing risk-premium variable) rather than at the input-forecasting method.

**(16)**

**(17) RMSE/MAE/MAPE summary table**

The same forecast-vs-hindsight comparison as before, reduced to headline error metrics:

| | RMSE | MAE | MAPE |
|---|---|---|---|
| Forecast vs. Real | 0.180 | 0.167 | 2.51% |
| Hindsight vs. Real | 0.197 | 0.184 | 2.75% |

The forecast's MAPE being lower than the hindsight version's reinforces the limitation discussed above.

**(18) Diagnostic tests: Jarque-Bera, Breusch-Pagan, VIF**

The three standard OLS-assumption checks on the final model:

- **Jarque-Bera (normality):** stat 3.55, p = 0.169 → fail to reject H0, residuals look normal.
- **Breusch-Pagan (heteroscedasticity):** LM stat 37.73, p ≈ 0.000 → reject H0, evidence of heteroscedasticity.
- **VIF (multicollinearity):** `m3_growth` 1.45, `us_10y` 6.39, `sofr` 20.56, `cpi` 2.60, `wpi` 15.35.

Normal residuals is good news for the validity of the t-tests/p-values reported throughout — no need to caveat coefficient significance on non-normality grounds. The heteroscedasticity result means the standard errors reported are technically not fully reliable (OLS assumes constant residual variance to get correct standard errors) — practically, this means some p-values may be slightly overstated or understated in precision, though it doesn't bias the coefficient point estimates themselves. A VIF above ~10 is conventionally considered severe; SOFR's VIF of 20.6 and WPI's of 15.4 are both well past that line, meaning SOFR, WPI, and (implicitly) the dropped `repo_rate` and `us_10y` are all substantially explaining each other. This directly supports the finding that it's not that the domestic policy rate doesn't matter to the 10Y yield — it's that its effect can't be cleanly separated from SOFR and WPI's overlapping information with this data and this specification.

**(19) Final model fit summary + coefficient significance table**

A clean restatement of the final model's headline statistics (R² 0.7379, Adj. R² 0.7321, F = 128.37, p ≈ 0) and a formatted significance table confirming all five coefficients (`m3_growth`, `us_10y`, `sofr`, `cpi`, `wpi`) are significant at the 5% level, alongside the jointly-significant F-test.

---

## 7. Limitations

Four limitations stand out from the diagnostic and validation work above.

**Residual autocorrelation.** The Durbin-Watson statistic for the final model is 0.146, well below the range associated with independent residuals. This indicates the model's errors are correlated from one week to the next — given the specification uses only current-period macro levels and has no mechanism to account for the yield's own recent momentum. Practically, this means the standard errors and p-values reported for the model's coefficients should be treated as approximate rather than exact; the coefficients' signs and relative magnitudes remain informative, but their precision is somewhat overstated. This is a well-understood and common feature of static levels regressions on financial time series, and it points directly to the most natural extension of this work: incorporating a lagged-yield term to absorb the yield's own persistence, which was tested in a preliminary form during this project and is a clear candidate for the next iteration.

**Heteroscedasticity.** A Breusch-Pagan test rejects the null of constant residual variance (p≈0.0000), meaning the size of the model's typical error is not uniform across the sample. As with autocorrelation, this affects the reliability of standard errors rather than the coefficient estimates themselves, and is addressable with heteroscedasticity-robust standard errors (e.g., HC3), a standard and low-cost correction that does not require reworking the model's specification.

**Multicollinearity among rate variables.** SOFR and WPI both show high variance inflation factors (20.6 and 15.4 respectively), reflecting real, expected overlap between the US policy rate proxy, the US 10Y yield, and inflation-linked variables. This means individual coefficients should be interpreted as representing a broader rate/liquidity or inflation channel rather than a fully isolated, independent effect — for example, SOFR's coefficient captures much of the "global monetary conditions" channel that a less collinear specification might have split across SOFR, US 10Y, and the domestic repo rate. This was tested directly rather than left as a caveat: a reduced specification without SOFR produced meaningfully worse in-sample and out-of-sample performance, confirming that despite the collinearity, SOFR carries genuine explanatory value the remaining variables do not fully substitute for.

**A consistent, one-directional forecast bias.** Across every out-of-sample check performed — three individually verified 2026 dates and a full Q1 2026 trend-extrapolated forecast — the model underpredicted the realized yield, never overpredicted, by a margin that narrowed over time (from roughly −3.1% in February to −1.5% in April, and from a Q1 average of −16.7bps to single-digit basis points by late March). Decomposing this error further shows the gap originates primarily in the model specification itself rather than in the macro-forecasting method: a version of the model fed the true, realized macro inputs (rather than trend-extrapolated ones) produced a similar or slightly larger error than the forecast did. This points toward a real, missing structural factor — plausibly a term or risk premium, fiscal and auction-supply pressure, or global risk sentiment — operating on the 10Y yield independently of the five variables currently in the model, rather than a shortcoming in how those five variables were projected forward. Identifying and incorporating such a factor is the most promising direction for improving the model's accuracy going forward.

Taken together, these limitations do not undermine the model's core finding — that a small set of economically grounded macro variables explains a substantial share (R²=0.74) of the India 10Y yield's movement, with statistically and economically sensible coefficients — but they do mean the model is best understood as a solid, interpretable baseline rather than a finished, precision forecasting tool. Each limitation identified here was diagnosed through explicit testing rather than assumed, and each has a concrete, actionable next step, which is the standard this analysis has aimed to hold itself to throughout.

---

## 8. Conclusion

This project models the Indian 10-Year G-Sec yield using roughly 4.5 years of RBI/CCIL trade data and seven macro-financial variables — repo rate, M3 money supply growth, US 10Y Treasury yield, SOFR, USD/INR, CPI, and WPI — drawing on the established framework of Dua and Raje's (2014) work on Indian government security yield determinants. The analysis proceeded through several iterations: an initial Principal Component Analysis decomposed the macro panel into a small number of factors, one of which (PC2) showed a strong standalone correlation with the yield (−0.88) even after correcting for target leakage; subsequent ARIMA testing on these factors, however, found no exploitable time-series structure — both PC1 and PC2 were statistically indistinguishable from a random walk — leading to a pivot toward direct regression.

During this process, a critical data integrity issue was uncovered: CPI and WPI were each found frozen at a single value for 30+ consecutive weeks, traced to unreconciled base-year revisions in the source data, and resolved by sourcing updated, methodologically consistent datasets for both series.

With corrected data, backward elimination and an independent forward stepwise procedure both converged on an identical five-variable specification — M3 growth, US 10Y, SOFR, CPI, and WPI — explaining 73.8% of yield variance, with every coefficient statistically and economically significant in terms of sign and magnitude. The model was validated genuinely out-of-sample in two ways: against three individually verified real-world 2026 dates, and against a full quarter (Q1 2026) forecast built entirely from macro inputs trend-extrapolated using only pre-2026 data. The latter produced a mean absolute error of approximately 17 basis points (MAPE 2.5%) against real, subsequently-released yield data.

Diagnostic testing surfaced a consistent, one-directional forecast bias (the model systematically underpredicts, though the gap narrows over time), residual autocorrelation (Durbin-Watson = 0.146), heteroscedasticity, and meaningful multicollinearity between SOFR and related rate variables, each investigated directly rather than left unexamined, including an explicit test confirming SOFR's collinear-but-genuine contribution to model accuracy.
