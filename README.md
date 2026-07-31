# SQL Database Creation - Portfolio Analysis

## Project goal

This project is a database engineering exercise, not a portfolio optimization tool. The goal was to design, build, and populate a relational database from scratch in MySQL: a normalized schema, primary and foreign keys, and parameterized stored procedures that actually work end to end.

The portfolio return, risk, and Sharpe ratio analysis is included because the assignment required it to be built in SQL, but this is not how the analysis would be done in practice. In a real workflow, portfolio optimization, covariance matrices, and efficient frontier calculations would be done in Python (numpy, pandas, scipy.optimize, PyPortfolioOpt) or R (PerformanceAnalytics, quantmod), which are built for numerical and statistical computation in a way SQL isn't. 

A relational database built from scratch in MySQL, designed around a fictional Ultra High Net Worth client (Palo Alto, CA) holding a $95M five-ETF portfolio. Built for an Individual Final Assessment in SQL & Data Management.

## Client Portfolio

| Ticker | Name | Allocation | Asset Class |
|--------|------|-----------:|-------------|
| IXN | iShares Global Tech ETF | 17.5% | Equity |
| QQQ | Invesco QQQ Trust (NASDAQ 100) | 22.1% | Equity |
| IEF | iShares 7-10 Year Treasury Bond | 28.5% | Fixed Income |
| VNQ | Vanguard Real Estate ETF | 8.9% | Real Assets |
| GLD | SPDR Gold Shares | 23.0% | Commodities |

## Analysis performed

All five questions below are answered through parameterized MySQL stored procedures, with the time window passed in as an input parameter, covering the period 2023-06-16 to 2026-06-15.

1. Returns (`q1_returns`): 12M, 18M, 24M total return per holding and for the whole portfolio.
2. Correlations (`q2_correlations`): Pearson correlation between all asset pairs, computed manually since `CORR()` is unavailable in this MySQL version, across 6M to 24M windows.
3. Volatility (`q3_risk`): annualized sigma per holding and for the portfolio, 6M and 12M windows.
4. Rebalancing recommendation (`q_sharpe`): Sharpe ratio (RF = 3.68%, 3M U.S. T-Bill) for every current holding, ranking risk adjusted performance to drive sell and buy decisions.
5. Optimized portfolio impact (`new_sharpe`): Sharpe ratio, return, and volatility of the rebalanced portfolio after selling QQQ, trimming IEF, GLD, and VNQ, and adding AVDV, VTIP, and DBC.

### Key result

| Metric | Current | Optimized | Change |
|--------|--------:|----------:|-------:|
| Annualized return (36M) | 19.58% | 19.11% | -0.47pp |
| Volatility (12M) | 12.24% | 10.89% | -1.35pp |
| Sharpe ratio (36M) | 1.367 | 1.531 | +0.164 |

Full methodology, correlation heatmaps, trade plan, and citations are in the [PDF report](Client_Report_PortfolioAnalysis.pdf).

## Data pipeline

Daily adjusted close pricing was sourced via the yfinance Python API (`auto_adjust=False`), producing long format CSVs, then loaded into MySQL Workbench through generated INSERT statements. Holding quantities are derived from position value divided by live market price at the reporting date.

## Tools

- MySQL 8 / MySQL Workbench
- Python (yfinance) for data download
- Python (matplotlib, pandas) for chart generation in the report

## Scope and limitations

- Database first, not a trading system. The goal was a normalized schema (`customer_details`, `accounts_dim`, `security_masterlist`, `holdings_dim`, `pricing_daily_new`) and stored procedures that take the time window as an input parameter, not a production analytics pipeline.
- SQL is not the natural tool for portfolio optimization. There is no native `CORR()` support in this MySQL version, worked around manually, no covariance matrix or matrix algebra support, and no built in solver for mean variance optimization or an efficient frontier. The optimized portfolio here is a small, manually chosen set of candidate weights evaluated through the Sharpe ratio, not the output of a true optimizer such as `scipy.optimize.minimize` or PyPortfolioOpt's `EfficientFrontier`.
- Static universe. Candidate additions (AVDV, VTIP, DBC) were hand picked based on Sharpe ranking and qualitative diversification reasoning, not screened programmatically from a broader universe.
- Fictional client, academic exercise. All client data, holdings, and recommendations are for coursework only and are not real investment advice.

## Skills demonstrated

- Relational schema design with primary and foreign keys across 5 tables
- Parameterized MySQL stored procedures, with IN parameters driving dynamic date window logic via DATE_SUB
- Window functions (FIRST_VALUE, LAG) for time series return and volatility calculations
- Manual statistical computation in SQL (Pearson correlation, annualized standard deviation, Sharpe ratio) where native functions were unavailable
- ETL pipeline: API to CSV to SQL INSERT (yfinance to convertcsv.com to MySQL Workbench)
