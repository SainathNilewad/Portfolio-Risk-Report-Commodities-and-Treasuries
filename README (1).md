# Portfolio Risk Report: Commodities and Treasuries

## 1. Project Overview

A beginner-friendly market-risk analysis of an equal-weighted portfolio of **WTI crude oil, gold and the IEF Treasury bond ETF** (1/3 each, USD 1,000,000, rebalanced daily to fixed weights), plus a short fixed-income section on the Treasury yield curve.

> **Question it answers:** "How much could this portfolio lose in a bad day, how do we know that number is reliable, and what happens in a crash?"

| Part | What it does |
|---|---|
| **A. Returns and portfolio** | Daily returns, portfolio return, volatility and tail statistics, correlation heatmap and 60-day rolling correlations |
| **B. VaR and Expected Shortfall** | 1-day Historical, Parametric (normal) and Monte Carlo VaR at 95% and 99%; Expected Shortfall; 10-day scaling |
| **C. VaR backtest** | Rolling 250-day VaR with no look-ahead; breach counts tested with a z-score and the Kupiec test |
| **D. Stress tests** | COVID crash window, 2022 rate-hike year, 10 worst days, two hypothetical shock scenarios |
| **E. Fixed income** | 10Y minus 2Y yield-curve slope and inversions; IEF sensitivity to yield changes |

> **Important:** This is an educational historical analysis, not investment advice. WTI is a **spot** series, gold is a **continuous futures** series and IEF is an ETF; none is a modelled tradable position, and weights are fixed with no transaction costs.

## 2. Data

Put these five files in a `data/` folder next to the notebook.

| File | Content | Source |
|---|---|---|
| `wti.csv` | WTI spot price, USD per barrel (`observation_date`, `DCOILWTICO`) | FRED `DCOILWTICO`: https://fred.stlouisfed.org/series/DCOILWTICO |
| `gold.csv` | Daily gold price, USD per troy ounce (`date`, `price`) | Yahoo Finance ticker `GC=F`: continuous front-month COMEX gold futures, daily close. **Not the LBMA fixing**: dates follow the US exchange calendar. Continuous futures series can contain small gaps at contract rolls |
| `ief.csv` | IEF (7-10 year Treasury ETF) adjusted close (`date`, `price`) | Yahoo Finance, `Adj Close`, as downloaded. Adjusted close is used because the fund pays regular distributions |
| `dgs10.csv` | 10-year Treasury yield, percent (`observation_date`, `DGS10`) | FRED `DGS10`: https://fred.stlouisfed.org/series/DGS10 |
| `dgs2.csv` | 2-year Treasury yield, percent (`observation_date`, `DGS2`) | FRED `DGS2`: https://fred.stlouisfed.org/series/DGS2 |

- **Two aligned tables.** The portfolio uses WTI, gold and IEF only (3,735 days, 21 Oct 2011 to 14 Oct 2026). The yields have no values on US bond-market holidays, so they are joined separately for Part E only (3,741 days). Joining everything together would drop valid portfolio days.
- **April 2020:** WTI spot was negative on 20 April 2020, so percentage returns are undefined. WTI returns are set to zero on that day and the next. This is a **modelling assumption** that understates tail risk.

## 3. Tools

Python, Jupyter, pandas, NumPy, Matplotlib, seaborn.

```bash
pip install pandas numpy matplotlib seaborn
```

## 4. How to Run

1. Put the five CSV files in `data/` (rename `gold_1.csv` and `wti_1.csv` to `gold.csv` and `wti.csv` if needed; do not mix them with older files of the same name from other projects).
2. Open the notebook and choose **Run All**.
3. All settings (weights, confidence levels, window, simulations, stress windows, shock scenarios) are in one cell near the top.
4. The last code cell runs 7 sanity checks, including a look-ahead test that changes one day's return and confirms no earlier VaR cutoff moves.

## 5. Results Snapshot

Data through October 2026. Re-run the notebook to reproduce, and do not quote these without checking.

**1-day loss estimates on USD 1,000,000**

| Method | 95% | 99% |
|---|---|---|
| Historical VaR | $14,089 | $24,757 |
| Parametric VaR | $17,205 | $24,469 |
| Monte Carlo VaR | $17,237 | $24,292 |
| Expected Shortfall (historical) | $22,348 | $37,689 |

**VaR backtest (rolling 250 days, 3,484 days tested)**

| Level | Breaches | Expected | z-score | Kupiec LR | Verdict |
|---|---|---|---|---|---|
| 95% | 192 | 174.2 | 1.38 | 1.86 | Consistent with the model |
| 99% | 50 | 34.8 | 2.58 | 5.87 | Too many breaches (risk understated) |

**Stress and rates:** COVID window -20.2% (-$202,141); 2022 rate-hike year -0.9% (IEF fell 15.2% but WTI rose); 543 inverted-curve days (14.5%); IEF's estimated yield sensitivity is about -7.6% per 1 percentage-point rise in the 10-year yield.

## 6. Project Structure

```text
Portfolio-Risk-Report/
├── data/
│   ├── wti.csv
│   ├── gold.csv
│   ├── ief.csv
│   ├── dgs10.csv
│   └── dgs2.csv
├── Portfolio_Risk_Report_Commodities_and_Treasuries.ipynb
└── README.md
```

## 7. Limitations

- Mixed instruments: WTI spot, gold continuous futures (possible roll gaps), IEF ETF. No roll yield, margin or transaction costs.
- Fixed weights rebalanced daily; real portfolios drift.
- Normal-distribution assumption in Parametric and Monte Carlo VaR; the data has fat tails.
- One 15-year sample with a few extreme episodes (2020 dominates the worst days).
- April 2020 treatment understates the true loss a futures holder suffered.
- Backtest tests assume independent breaches, but breaches cluster in crises.
- The three assets are priced at slightly different times of day.
