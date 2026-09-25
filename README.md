# Portfolio optimization

My Python notebook for estimating portfolio weights that maximize the Sharpe ratio for URTH, BND, TTE, QQQ and EEM. It downloads adjusted prices, calculates annualized log-return estimates and covariance, optimizes weights with SciPy, and displays 5,000 simulated portfolios against the selected allocation.

## Run

1. Install dependencies: `pip install -r requirements.txt`.
2. Get a FRED API key and set the environment variable `FRED_API_KEY` before opening Jupyter. For example, in PowerShell: `$env:FRED_API_KEY="your-key"`; in macOS/Linux: `export FRED_API_KEY="your-key"`. Never commit your key.
3. Run `jupyter notebook PortfolioOptimization.ipynb`, then run all cells.

The notebook fetches the latest 3-month US Treasury bill quote (`DTB3`) and current market data; outputs therefore change over time. The raw DTB3 quote is an annual discount-basis rate and an approximation to an investable risk-free return. Mean log returns are annualized with 252 trading days. The optimization is in-sample and excludes costs; the simulated points are not the efficient frontier. This is an educational project, not investment advice.

## Small corrections before publication

The API key was moved into an environment variable. The random portfolio loop now rejects weights above the existing 80% cap after normalization. The prior chart output was cleared because it was produced before this correction. The underlying calculations, functions and chart otherwise remain the author's original code.
