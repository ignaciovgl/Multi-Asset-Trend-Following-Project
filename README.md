# Multi-Asset Trend-Following Backtest

A systematic trend-following strategy built from scratch — data pipeline, signal construction, volatility-targeted position sizing, and a robustness-tested backtest across equities, bonds, FX, and commodities (2000–present).

Built as a quant research portfolio project in Google Colab.

## Summary

- **Universe:** 14 instruments across 4 asset classes (equity indices, government bonds, FX majors, commodities), sourced via a hybrid of free continuous futures and liquid ETF proxies where futures data wasn't freely available
- **Signals:** 3 complementary trend signals (EWMA crossover, 12-1 month time series momentum, Donchian channel breakout), normalised and equally weighted into a composite
- **Sizing:** instrument- and portfolio-level volatility targeting, position caps, transaction costs, T+1 execution
- **Backtest:** in-sample (2000–2015) / out-of-sample (2015–present) split, with sensitivity analysis, stress testing, and a 60/40 benchmark comparison

**Headline result:** the strategy achieved a full-period Sharpe of 0.42 (in-sample 0.58, out-of-sample 0.23) — comparable risk-adjusted return to a 60/40 benchmark (Sharpe 0.35), but with a substantially shallower max drawdown (-21.4% vs -36.6%) and a mildly negative correlation to it (-0.109), indicating a genuine diversification benefit rather than replicated market beta.

## Repository structure

```
├── Multi_asset_trend_following_backtest_project.ipynb   # Full pipeline, end to end
├── data/
│   ├── raw/            # Pulled prices, instrument metadata, coverage log
│   └── processed/      # Returns, vol, correlations, signals, positions, performance tables
├── report/
│   └── report.pdf       # 4-6 page written report: methodology, results, limitations
└── README.md
```

## Methodology

### 1. Data

Prices are pulled via `yfinance` (equities, FX, real futures) and ETF proxies (Bund, Gilt) where free continuous futures data isn't available. Each instrument enters the backtest on its own native start date rather than being truncated to a common window — this preserves 20+ years of history, including the 2008 GFC, for instruments like equities and commodities that would otherwise be discarded to accommodate shorter-history additions like the Bund proxy (2012). Returns are computed as log returns throughout, for time-additivity and consistent behaviour across instruments of very different price levels and volatility.

### 2. Signals

Three signals, each capturing a different trend mechanism:
- **EWMA crossover** — fast vs. slow exponentially-weighted moving averages, multiple horizon pairs averaged together
- **Time series momentum (12-1 month)** — trailing 12-month return, excluding the most recent month to avoid short-term mean-reversion effects
- **Donchian channel breakout** — price position within its recent high-low range

Each is normalised to [-1, +1] via rolling z-score and `tanh`, then combined into an equal-weighted composite.

### 3. Position sizing

Positions are sized in risk terms rather than notional terms:
- **Instrument-level:** scaled by realised volatility so a full-conviction signal contributes a fixed 1% daily volatility, regardless of asset class
- **Portfolio-level:** leverage scaled dynamically so realised portfolio volatility (which accounts for cross-instrument correlation) targets 10% annualised
- **Position cap:** 20% notional per instrument, as a backstop against stale volatility estimates and unrealistic liquidity assumptions
- **Costs:** turnover-based transaction costs, differentiated by instrument liquidity (0.1bps futures/FX, 0.5bps ETF-sourced)
- **Execution:** T+1, avoiding look-ahead bias

### 4. Backtest & robustness

- In-sample (2000–2015) vs out-of-sample (2015–present) split
- Sensitivity sweeps on position cap, instrument vol target, and EWMA signal speed
- Stress tests on 2008 GFC, 2020 COVID, and 2022 inflation shock
- Benchmark comparison against a static 60/40 (SPX/US10Y) portfolio

## Key results

| | Annualised Return | Annualised Vol | Sharpe | Calmar | Max Drawdown | DD Duration (days) |
|---|---|---|---|---|---|---|
| In-Sample (2000-2015) | 6.26% | 10.46% | 0.58 | 0.33 | -19.2% | 1098 |
| Out-of-Sample (2015-present) | 2.57% | 10.88% | 0.23 | 0.12 | -21.4% | 1068 |
| **Full Period** | **4.63%** | **10.65%** | **0.42** | **0.22** | **-21.4%** | **1098** |
| 60/40 Benchmark | 3.89% | 10.93% | 0.35 | 0.11 | -36.6% | 1597 |

**By asset class (full period Sharpe):** equity 0.33, commodity 0.29, FX 0.23, bond -0.06. The bond sleeve underperforms, plausibly linked to its shorter, ETF-proxy-sourced history (Bund from 2012, Gilt from 2008) rather than a signal failure.

**Sensitivity:** volatility target and position cap are both low-sensitivity parameters (Sharpe range <0.07 across wide sweeps). EWMA signal speed is not — fast lookbacks underperform materially (Sharpe 0.23, max drawdown -34%) relative to the base configuration used throughout, which was chosen a priori on convention rather than fit to this result.

**Stress periods (full-period Sharpe):** 2008 GFC 0.95, 2020 COVID 2.24, 2022 inflation shock 0.60 — consistent with trend-following's expected strength in sustained directional macro shocks, though a finer conditional analysis showed the edge over the 60/40 benchmark specifically narrowing during 2022-2023, in line with trend-following's known difficulty in fast, correlated regime shifts.

Full tables and charts are in the notebook and report.

## Limitations

- **Data:** commodity and US10Y series use unadjusted continuous futures (roll-yield gaps not corrected for); Bund and Gilt use ETF proxies (expense-ratio drag, shorter history) in place of futures data that isn't freely available.
- **Staggered universe:** the tradeable instrument set grows over time as later-starting instruments (EURUSD/GBPUSD from 2003, AUDUSD from 2006, Gilt from 2008, Bund from 2012) become available — early-period and late-period results aren't on a fully like-for-like universe.
- **Signal speed sensitivity:** performance is more sensitive to EWMA lookback choice than to vol target or position cap — see sensitivity analysis.
- **Risk-free rate:** Sharpe ratio uses a 0% risk-free rate throughout, a simplification given the backtest spans both near-zero and 4%+ rate regimes.

## Running this project

1. Open the notebook in Google Colab
2. Mount Google Drive and set `PROJECT_DIR`
3. Add a free [FRED API key](https://fred.stlouisfed.org/docs/api/api_key.html) if extending the rates data
4. Run top to bottom — Phase 1 (data) through Phase 5 (robustness testing)

## Author

Ignacio Vigil — [github.com/ignaciovgl](https://github.com/ignaciovgl)
