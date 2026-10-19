# Semiconductor Pairs Trading | Cointegration and Out-of-Sample Backtesting

An end-to-end quantitative finance project that screens semiconductor-related stocks for cointegration, estimates a mean-reverting spread, and evaluates a systematic long/short trading strategy **out of sample**.

The research begins with 25 stocks, applies unit-root and Engle–Granger cointegration tests, and ultimately selects **AMAT (Applied Materials) and NXPI (NXP Semiconductors)** for the trading backtest.

> **Notebook:** [Semiconductor_Pairs_Trading_EN.ipynb](Semiconductor_Pairs_Trading_EN.ipynb)  
> **Research period:** January 2021 onward | **Training:** 2021–2024 | **Backtest:** January 2025–October 2026  
> **Saved results:** through **October 16, 2026**. A fresh data download may yield different results.

## Research objectives

1. Identify economically related stocks using correlations in **daily log returns**.
2. Check whether log prices are consistent with integration of order one, \(I(1)\), using the Augmented Dickey–Fuller (ADF) test.
3. Test cointegration using **Engle–Granger**, accounting for multiple pair comparisons with a **Bonferroni correction**.
4. Assess stability across overlapping two-year windows and estimate mean-reversion speed using ARIMA/AR(1) models selected by **BIC**.
5. Backtest an explicitly defined Z-score strategy using parameters estimated **only on training data**.
6. Evaluate returns, transaction and borrowing costs, risk, exposure, and individual trades.

## Methodology

### 1. Screening and pair selection

- Download dividend- and split-adjusted daily prices with `yfinance` and normalize each stock to a base of 100.
- Use **2021–2024** as the in-sample estimation window.
- Calculate log prices and daily log returns; rank pairs by Pearson correlation of log returns.
- Run ADF tests on levels and first differences. Of 25 initial stocks, **24** are classified as consistent with \(I(1)\) under the notebook's screening rule.
- Test the resulting **276 pairs** with Engle–Granger. **AMAT–NXPI** and **LRCX–NXPI** pass the Bonferroni-adjusted 5% threshold.
- Select AMAT–NXPI because the Engle–Granger test also indicates cointegration in each of the three examined two-year windows (2021–2022, 2022–2023, and 2023–2024).

### 2. Spread and mean reversion

The estimated training-period relationship, using normalized prices, is:

$$
\ln(P_{NXPI,t}) = 1.7689 + 0.6051\ln(P_{AMAT,t}) + \varepsilon_t
$$

The spread is the estimated residual:

$$
S_t = \ln(P_{NXPI,t}) - \hat\alpha - \hat\beta\ln(P_{AMAT,t})
$$

Its Z-score uses the **training-period** residual mean and standard deviation, which remain fixed in the backtest. A BIC-selected AR(1) model fitted to the training residuals implies a mean-reversion half-life of approximately **9.8 trading sessions**.

### 3. Trading and execution rules

| Rule | Definition |
|---|---|
| Long spread | Z-score crosses below **-2** (but remains above -4): buy NXPI, short AMAT |
| Short spread | Z-score crosses above **+2** (but remains below +4): short NXPI, buy AMAT |
| Take profit | Z-score crosses zero in the favorable direction |
| Stop-loss | Adverse Z-score reaches **-4** (long spread) or **+4** (short spread) |
| Maximum holding period | **30 sessions** |
| Stop-loss lockout | Do not reopen until the absolute Z-score falls below **1** |
| Execution | Signal computed at the close; assumed fill at the **next session's close** |
| Trading costs | **0.05%** per unit of portfolio turnover |
| Short-borrowing cost | **3% annualized** on the short-leg weight |

Positions are scaled so the **sum of absolute leg weights is one**. This is gross-exposure normalization, **not necessarily dollar neutrality**. Returns are recorded from the session after the assumed execution close.

## Saved out-of-sample results

Results below are from the notebook's saved execution covering **January 2025 to October 16, 2026**, with the original adjusted-data snapshot and the simplified cost model.

| Performance measure | Result |
|---|---:|
| Total gross return | 42.74% |
| **Total net return** | **40.90%** |
| Annualized return (CAGR) | 21.22% |
| Annualized volatility | 11.93% |
| Sharpe ratio (risk-free rate = 0) | 1.67 |
| Sortino ratio | 3.58 |
| Maximum drawdown | -5.92% |
| Calmar ratio | 3.59 |
| Time in market | 25.39% |
| Completed trades | 7 |
| Winning trades | 6 / 7 (85.71%) |
| Profit factor | 15.97 |
| Average winning trade | 6.33% |
| Average losing trade | -2.38% |
| Average trade duration | 16.29 sessions |

Trade-level compounded returns reconcile to the daily strategy's **40.90% net cumulative return**. No position is open at the end of the saved backtest.

## Interpretation and limitations

The saved backtest produces attractive risk-adjusted results while being invested during only about a quarter of the observed sessions. However, **seven trades are far too few** to establish that the performance is repeatable. The strategy records a stop-loss exit in October 2025 and makes no later entries in the saved sample; the out-of-sample spread subsequently deviates persistently from its historical equilibrium. This is **suggestive of instability**, not a formal finding that cointegration has broken down.

Important limitations:

- Parameter estimation and strategy selection rely on one historical training period, even though initial pairwise cointegration screening accounts for multiple comparisons.
- Borrowing and trading costs are simplified; **slippage, market impact, taxes, financing/margin requirements, and borrow availability** are not simulated.
- A **Z-score stop-loss is not a guaranteed monetary loss limit**, particularly with delayed execution.
- Portfolio weights normalize gross exposure, but do not imply zero net dollar exposure.
- Prices are downloaded using a **dynamic end date** and may be revised by the data provider. Results will not be exactly reproducible without freezing the input data and cutoff date.
- The notebook preserves its original saved charts and outputs. Some cached plot annotations reflect the original Spanish labels; rerunning the translated notebook refreshes display labels in English.

The project is an **educational backtesting exercise**, not evidence of a deployable or profitable live strategy.

## Installation and execution

Requirements: **Python 3.10+**, internet access for Yahoo Finance data, and the following libraries:

```bash
pip install numpy pandas yfinance matplotlib seaborn statsmodels notebook
```

Clone the repository and open the notebook:

```bash
jupyter notebook Semiconductor_Pairs_Trading_EN.ipynb
```

Run cells **in order from the beginning**. The notebook downloads data dynamically through `yfinance`; its saved output reflects a previous run and may not reproduce exactly with later prices, revisions or package versions. No API keys or proprietary datasets are required.

**Code compatibility:** Some variable names and DataFrame columns remain in Spanish to preserve the original working implementation, for example `Posicion` (executed position), `Posicion_efectiva` (effective P&L position), `Rentabilidad_Neta` (daily net return), and `Motivo_salida` (exit reason). Explanations, comments, and newly generated chart labels are in English.

## Tools and concepts

**Python:** NumPy, pandas, yfinance, Matplotlib, Seaborn, statsmodels, Jupyter.  
**Quantitative methods:** ADF unit-root testing, Engle–Granger cointegration, OLS hedge-ratio estimation, ARIMA/AR(1) modeling, rolling-window diagnostics, half-life estimation, Z-score trading, out-of-sample backtesting, Sharpe/Sortino, drawdown, and trade-level attribution.

---

*For educational and research purposes only. Not investment advice.*
