# Portfolio Optimization with the Sharpe Ratio

This project explores how to allocate capital across five assets to maximize the **estimated Sharpe ratio**. It applies the mean–variance framework to historical market data, solves a constrained optimization problem, and compares the result with 5,000 randomly generated portfolios.

The notebook is my implementation of the model, from data collection to visualization.

## Research question

> Given the historical returns and correlations of URTH, BND, TTE, QQQ and EEM, which long-only allocation offers the highest estimated return per unit of volatility, with no asset representing more than 80% of the portfolio?

The five instruments give the portfolio exposure to global equities, US bonds, a single energy company, US growth/technology and emerging markets. The mix is illustrative, not an assertion that these are the best assets to hold.

## Workflow

1. Download adjusted closing prices for the five tickers from Yahoo Finance over approximately 13 years.
2. Compute daily log returns and estimate annualized mean returns and the covariance matrix.
3. Obtain a US three-month Treasury bill rate (`DTB3`) from FRED as the risk-free-rate proxy.
4. Maximize the estimated Sharpe ratio using constrained optimization (`scipy.optimize.minimize`, SLSQP).
5. Generate 5,000 random portfolios that obey the same 80% individual-weight limit and compare their estimated risk and return with the optimum.
6. Plot the portfolios and an illustrative line through the risk-free-rate intercept and the optimized portfolio.
7. !!! The 80% limit for bounds is arbitrary, we can choose another one limit to see the effect on the optimal weights and on the Sharpe Ratio

## Mathematical model

Let $$P_{i,t}$$ be the adjusted closing price of asset $$i$$ on day $$t$$. Its daily **log return** is

$$
r_{i,t}=\ln\left(\frac{P_{i,t}}{P_{i,t-1}}\right).
$$

For $$N=5$$ assets, write the portfolio weights as $$w=(w_1,\ldots,w_N)^\top$$. The notebook estimates the vector of annualized mean log returns and the annualized covariance matrix as

$$
\hat{\mu} = 252 \overline{r}
$$

$$
\hat{\Sigma} = 252 \mathrm{Cov}(r)
$$

Using these estimates, it calculates the portfolio's **estimated annual log return** and **annualized volatility**:

$$
\hat{\mu}_p(w)=w^\top\hat{\mu},
\qquad
\hat{\sigma}_p(w)=\sqrt{w^\top\hat{\Sigma}w}.
$$

The optimization objective is

$$
\max_w\quad
\hat{S}(w)=\frac{\hat{\mu}_p(w)-r_f}{\hat{\sigma}_p(w)}
\qquad\text{subject to}\qquad
\sum_{i=1}^{N}w_i=1,
\quad 0\leq w_i\leq 0.80.
$$

Here $$r_f$$ is the annualized risk-free-rate proxy. The sum constraint invests the entire portfolio; the bounds prohibit short sales and cap any single position at 80%. The code minimizes **negative Sharpe** because SciPy's `minimize` solves minimization problems.

**Return convention:** The notebook uses log returns for the assets but takes the FRED bill quote directly as $$r_f$$. It is a practical approximation: `DTB3` is quoted on a discount basis, not as an annual log return. For a more exact comparison, convert the Treasury quote to an investment return and then to a log return on a consistent horizon. Also, the weighted sum of asset log returns is an approximation to the return on a discretely rebalanced portfolio; for an exact daily portfolio return, use weighted **simple** asset returns.

## Interpreting the visualization

- Each colored point represents a randomly generated feasible allocation. Its position shows estimated annual volatility (horizontal axis) and estimated annual log return (vertical axis); its color shows the estimated Sharpe ratio.
- The red star is the allocation returned by SLSQP. Its estimated Sharpe ratio is printed in the notebook.
- The dashed line starts at the risk-free-rate proxy and has the optimized portfolio's estimated Sharpe ratio as its slope. It is illustrative: mixing with cash or borrowing to follow the full line would require assumptions beyond the long-only, fully invested risky-asset optimization.

The cloud of random points **is not the efficient frontier**. A proper frontier would require solving a separate optimization problem for a range of target returns.

## Run the notebook



I put my own API key but the FRED request needs your own API key. Set it as an environment variable **before launching Jupyter**:



## Limitations and next steps

- **In-sample optimization:** The same historical observations estimate the inputs and determine the weights. A high fitted Sharpe ratio does not demonstrate future performance. A useful next step is to fit on an earlier period and assess the portfolio on a later, untouched period.
- **Cash-rate timing:** The notebook uses the latest `DTB3` observation alongside roughly 13 years of historical asset returns. For a historical performance analysis, use risk-free returns aligned with the dates of the asset returns.
- **Estimation risk:** Small changes in sample period, expected returns or covariances can produce materially different optimal weights. The result also depends on the selected five assets and the 80% cap.
- **Implementation costs:** Transaction costs, taxes, market impact and the effects of rebalancing are not included.
- **Annualization:** Multiplying daily mean returns and covariance by 252 assumes 252 trading days per year; this is a convention, not a prediction.

## Data and tools

| Purpose | Source or library |
| --- | --- |
| Adjusted market prices | Yahoo Finance via `yfinance` |
| US three-month Treasury bill quote | FRED series [`DTB3`](https://fred.stlouisfed.org/series/DTB3) via `fredapi` |
| Data and numerical calculations | `pandas`, `numpy` |
| Constrained optimization | `scipy.optimize` (SLSQP) |
| Visualization | `matplotlib` |

