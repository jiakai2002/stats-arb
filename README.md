# Statistical Arbitrage — Pairs Trading Framework

A pairs trading backtester built in Python, comparing three hedge ratio estimation methods across a broad universe of cointegrated equity pairs.

## Overview

The strategy screens a candidate universe for cointegrated pairs, models the spread, and generates mean-reversion signals from the standardised z-score. Three hedge ratio models are benchmarked out-of-sample across all qualifying pairs.

**Universe:** 13 ETF pairs across broad market, sector, commodities, and macro  
**Train period:** 2018–2019 | **Test period:** 2024

## Methodology

**Pair Selection**
- Engle-Granger cointegration test (p < 0.05)
- ADF test on spread (p < 0.05)
- Hurst exponent (H < 0.5 confirms mean-reversion)
- Half-life of mean reversion via OLS of lagged spread

2 of 13 candidate pairs passed all filters: **SPY/QQQ** and **QQQ/XLK**

**Hedge Ratio Estimation**
| Model | Description |
|---|---|
| Static OLS | Full-sample OLS regression, fixed beta |
| Rolling OLS | 60-day rolling window OLS, adaptive beta |
| Kalman Filter | State-space model with continuous Bayesian updates |

**Signal Generation**
- Z-score computed over a 20-day rolling window
- Entry at ±2σ, exit at ∓0.5σ

**Backtest**
- 10bps round-trip transaction cost
- Volatility targeting at 15% annualised (20-day realised vol, capped at 5×)
- Position scaling applied with a 1-day lag
- Metrics: Sharpe, annualised PnL, max drawdown, win rate, trade count, avg hold (days)

## Results (Out-of-Sample 2024, Top Performers)

| Pair | Model | Sharpe | Ann. PnL | Max DD | Win Rate | Trades | Avg Hold |
|---|---|---|---|---|---|---|---|
| XLE/XOP | Kalman | 5.04 | 36.5% | -0.59% | 82.6% | 33 | 7.6d |
| SPY/GLD | Kalman | 4.10 | 35.9% | -1.61% | 78.0% | 28 | 9.0d |
| GDX/GDXJ | Kalman | 4.02 | 29.4% | -1.70% | 75.9% | 22 | 11.5d |
| SPY/QQQ | Kalman | 3.08 | 18.4% | -1.31% | 74.4% | 24 | 10.5d |
| SPY/QQQ | Static | 1.93 | 23.7% | -5.66% | 58.2% | 15 | 16.8d |

The Kalman filter dominates across all pairs, consistently delivering higher Sharpe and tighter drawdowns than Static or Rolling OLS — reflecting its ability to track non-stationary beta in real time.

## Stack

```
yfinance · pandas · numpy · statsmodels · pykalman · matplotlib
```

## Structure

```
stats_arb.ipynb    # Data → pair screening → hedge ratio modelling → backtest → results
```

---

## Limitations & Improvements

**1. Single out-of-sample window**  
Performance is evaluated on one fixed test period (2024), which may overstate robustness. A walk-forward or block cross-validation scheme across multiple periods would give a more reliable estimate of strategy consistency.

**2. Simplified transaction costs**  
The backtest applies a flat 10bps cost with no slippage or market impact. In practice, spread mean-reversion strategies can be sensitive to execution quality, especially at high vol-targeting scales. Adding bid-ask spread estimates and size-dependent impact would improve realism.

**3. Static pair selection**  
Pairs are screened once on training data. Cointegration relationships break down over time — particularly during regime shifts — but the strategy has no mechanism to detect this. Periodic re-screening or a rolling cointegration monitor would reduce the risk of trading structurally broken pairs.
