# SQL_DatabaseCreation_PortfolioAnalysis
MySQL relational database + parameterized stored procedures for portfolio risk/return analysis (SQL &amp; Data Management coursework)

**Simoncelli Capital - Portfolio Risk & Return Analysis**

SQL-based portfolio analysis system built for a fictional Ultra High Net Worth
client (Palo Alto, CA) holding a $95M five-ETF liquid portfolio. Built for an
Individual Final Assessment in SQL & Data Management.

## Client Portfolio

| Ticker | Name                              | Allocation | Asset Class   |
|--------|-----------------------------------|-----------:|---------------|
| IXN    | iShares Global Tech ETF           |      17.5% | Equity        |
| QQQ    | Invesco QQQ Trust (NASDAQ 100)    |      22.1% | Equity        |
| IEF    | iShares 7-10 Year Treasury Bond   |      28.5% | Fixed Income  |
| VNQ    | Vanguard Real Estate ETF          |       8.9% | Real Assets   |
| GLD    | SPDR Gold Shares                  |      23.0% | Commodities   |


## What's in this repo

```
├── sql/
│   └── simoncelli_capital_analysis.sql   # Full schema + 5 parameterized stored procedures
├── report/
│   └── Simoncelli_Capital_Portfolio_Analysis.pdf   # Final client-facing report (PDF)
├── data/
│   └── (raw price CSVs go here — see note below)
├── scripts/
│   └── (yfinance download script goes here — see note below)
└── README.md
```

## Analysis performed

All five questions below are answered via **parameterized MySQL stored
procedures** (time window passed in as an input parameter), covering the
period 2023-06-16 to 2026-06-15 (to avoid having huge datasets in local):

1. **Returns** (`q1_returns`) - 12M / 18M / 24M total return per holding and
   for the whole portfolio.
2. **Correlations** (`q2_correlations`) - Pearson correlation between all
   asset pairs (computed manually in SQL since `CORR()` is unavailable in
   this MySQL version), 6M through 24M windows.
3. **Volatility** (`q3_risk`) - annualized sigma per holding and for the
   portfolio, 6M and 12M windows.
4. **Rebalancing recommendation** (`q_sharpe`) - Sharpe ratio (RF = 3.68%,
   3M U.S. T-Bill) for every current holding, ranking risk-adjusted
   performance to drive sell/buy decisions.
5. **Optimized portfolio impact** (`new_sharpe`) - Sharpe ratio, return, and
   volatility of the rebalanced portfolio after selling QQQ, trimming IEF/
   GLD/VNQ, and adding AVDV, VTIP, and DBC.

### Key result

| Metric                  | Current | Optimized | Change  |
|--------------------------|--------:|----------:|--------:|
| Annualized return (36M) |  19.58% |    19.11% | -0.47pp |
| Volatility (12M)        |  12.24% |    10.89% | -1.35pp |
| Sharpe ratio (36M)      |   1.367 |     1.531 |  +0.164 |

Full methodology, correlation heatmaps, trade plan, and citations are in the
[PDF report](report/Simoncelli_Capital_Portfolio_Analysis.pdf).

## Data pipeline

Daily adjusted-close pricing was sourced via the [`yfinance`](https://pypi.org/project/yfinance/)
Python API (`auto_adjust=False`), producing long-format CSVs, then loaded
into MySQL Workbench via generated `INSERT` statements. Holding quantities
are derived from position value ÷ live market price at the reporting date.

## Tools

- MySQL 8 / MySQL Workbench
- Python (yfinance) for data download
- Python (matplotlib/pandas) for chart generation in the report

## Author

Individual assessment - not a team project.
