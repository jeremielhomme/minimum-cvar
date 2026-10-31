Minimum-CVaR portfolio allocation with GARCH and Monte Carlo

Objective - 
Find the allocation across five assets (AAPL, GLD, JPM, MSFT, XOM) 
that minimises the 1-day 99% Conditional Value at Risk (CVaR), and test whether
the risk forecasts hold up on days the model has not seen.


Why GARCH - 
A first version used constant volatility and historical correlations. 
Over a 10-year window, the simulated correlations were getting too far from the historical 
estimates. This suggested that the constant-volatility model was not adequately capturing 
changes in market conditions. Modelling volatility explicitly largely removed the problem.


Method -
1. Fit a GARCH(1,1) with Student's t innovations to each asset's daily log
   returns. Dividing returns by conditional volatility gives standardised
   residuals that are much less affected by volatility regimes.

2. Rank the residuals, convert them to normal scores, and then take the correlation of those. 
   Apply a Cholesky decomposition to generate correlated standard normals.

3. Map them to uniforms with the normal CDF, then to Student's t draws with the
   inverse CDF (ν fitted per asset, rescaled to unit variance) and scale by the
   next-day volatility forecast. This is a Gaussian copula with Student's t
   marginals: fat tails for each asset, but no extra tail dependence between
   assets.

4. Simulate 10,000,000 one-day portfolio scenarios and minimise the 99% CVaR
   with scipy.optimize (SLSQP; weights ≥ 0, summing to 1, no per-asset cap).


Validation - 
Backtest over the last 1,000 trading days. For each day the
GARCH and Student's t parameters and the correlation matrix are refitted on past data only,
and the 99% VaR forecast is compared with the realised return. Portfolio weights were 
optimised once using full-sample correlation estimates, introducing some look-ahead bias. 
The backtest is therefore only partially out-of-sample: daily risk forecasts use past data 
exclusively, but the portfolio allocation is not fully independent of the test period.
Result: 11 exceedances in 1,000 days (10 expected). Two days in a row exceeded the 
VaR and warrant further investigation.



Results - 
Equal-weight CVaR $23262 vs optimised $21677 (about 7% lower) on a
$1000000 portfolio, as of 27 october 2026. 
Optimal weights: [AAPL : 17.51%, GLD : 34.09%, JPM : 20.94%, MSFT : 9.68%, XOM : 17.79%].


Pipeline - 
Results are written to BigQuery and read by a Data Studio dashboard 
(https://datastudio.google.com/s/kQ4lT3ruJ00), so re-running the notebook refreshes the dashboard.