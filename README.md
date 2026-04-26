# market-microstructure-backtests

High-frequency market data research and backtesting project focused on short-horizon mean-reversion, market microstructure effects, and transaction-cost-aware strategy evaluation.

The project uses 1-second XAUUSD and crypto market data to study how prices behave after deviations from rolling moving averages. The goal is not to present a live trading system, but to build a serious research pipeline: data preparation, signal discovery, backtesting, cost sensitivity, walk-forward validation, and robustness checks.

---

## Project Overview

This project started with a simple research question:

> After price moves unusually far below a rolling moving average, does it tend to rebound over the next few minutes?

From there, the project developed into a full research workflow:

1. Load and clean high-frequency market data.
2. Convert raw data into efficient parquet files.
3. Compute rolling moving averages and deviation signals.
4. Measure conditional rebound distributions.
5. Build sequential backtests with realistic trade constraints.
6. Add transaction-cost assumptions.
7. Test fixed and flexible exit logic.
8. Use walk-forward train/validation/test splits.
9. Compare filtered strategies against no-filter and random baselines.
10. Run hostile robustness checks such as entry delay, exit delay, higher costs, and monthly breakdowns.

---
---

## Selected Figures

### Long-Horizon Equity Curve

![Long-Horizon Equity Curve](reports/figures/xauusd_LONGHORIZON_test_equity_cost1p0bps.png)

### Hostile Audit Equity Curve

![Hostile Audit Equity Curve](reports/figures/xauusd_HOSTILE_AUDIT_harsh_case_equity.png)

### Feature Importance

![Feature Importance](reports/figures/xauusd_strict_feature_importance.png)

### Cost Sensitivity

![Cost Sensitivity](reports/figures/xauusd_LONGHORIZON_test_cost_sensitivity.png)

---
## Data

The project currently focuses on:

- **XAUUSD 1-second market data**
- Selected crypto pairs for related research
- High-frequency OHLC-style time series
- Parquet-based storage for faster loading and analysis

Raw data is not included in this repository.

---

## Methods

### Signal Research

The core signal studied is deviation from a rolling moving average:

- MA300
- MA500
- percentage distance from moving average
- future maximum rebound over a fixed horizon
- future adverse movement over the same horizon

Example research question:

> If price is 0.10% below MA300, what is the distribution of the maximum rebound over the next 2,000 seconds?

This is used first as signal research, not as a direct trading rule.

---

### Sequential Backtesting

The project then converts signal research into sequential trade simulations.

Backtest rules include:

- one open trade at a time
- fixed take-profit
- stop-loss
- maximum holding time
- flexible take-profit variants
- cooldown logic
- transaction-cost assumptions
- train/test separation

Example exit types tested:

- fixed TP/SL
- protected flexible TP
- unprotected flexible TP
- time-based exit

---

## Machine Learning Filter

A later stage of the project uses labelled candidate events.

Future data is used only to label events during research, for example:

- future max rebound
- future max drawdown
- whether the event belongs to the highest-rebound group

The model then uses only entry-time features such as:

- deviation from MA300 / MA500
- recent short-horizon returns
- moving-average slope
- rolling volatility
- distance from recent highs/lows
- time-of-day features

The goal is to identify which deviation events are more likely to produce larger rebounds.

---

## Walk-Forward Validation

To reduce overfitting, the project uses strict time splits.

Example long-horizon setup:

- **Train:** 2023
- **Validation:** 2024
- **Final test:** 2025–2026

Strategy parameters are selected on the validation period and then evaluated on the final test period.

The project also compares:

- ML-filtered signals
- unfiltered baseline signals
- random same-count baselines

---

## Robustness Checks

Several hostile tests are included to check whether results depend on fragile assumptions:

- transaction costs from 1 to 10 bps
- fixed TP only
- protected flexible TP
- unprotected flexible TP
- entry delay
- exit delay
- extra slippage assumptions
- random-entry baselines
- monthly return breakdowns
- no-filter baseline comparison

These checks are important because high-frequency strategies can look strong in a notebook while being difficult to execute live.

---

## Example Research Findings

The initial moving-average deviation signal showed a clear conditional rebound structure: deeper deviations tended to produce larger average future rebounds.

However, frequent small trades were highly sensitive to transaction costs. This motivated a shift toward:

- fewer but higher-quality candidate events
- cost-aware filtering
- walk-forward validation
- robustness testing under harsher execution assumptions

Some historical backtests remained strong under multiple stress tests, but these results should be interpreted as research findings, not live trading claims.

---

## Tools Used

- Python
- pandas
- polars
- NumPy
- scikit-learn
- matplotlib
- Jupyter
- parquet

---

## Repository Structure

```text
market-microstructure-backtests/
│
├── README.md
│
├── notebooks/
│   ├── 01_data_loading_and_checks.ipynb
│   ├── 02_conditional_rebound_analysis.ipynb
│   ├── 03_sequential_backtest.ipynb
│   ├── 04_ml_event_filter.ipynb
│   └── 05_hostile_audit.ipynb
│
├── src/
│   ├── data_loading.py
│   ├── features.py
│   ├── backtest.py
│   ├── metrics.py
│   └── plotting.py
│
├── reports/
│   └── figures/
│
└── outputs/
