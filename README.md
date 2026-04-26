# market-microstructure-backtests

High-frequency market data research and backtesting project focused on short-horizon mean-reversion, market microstructure effects, and transaction-cost-aware strategy evaluation.

The project uses 1-second XAUUSD and crypto market data to study how prices behave after deviations from rolling moving averages. The goal is not to present a live trading system, but to build a serious research pipeline: data preparation, signal discovery, sequential backtesting, cost sensitivity, walk-forward validation, machine-learning-based event filtering, and robustness checks.

> **Note:** This is a research project, not a live trading system. Historical backtest results are not live trading claims. Execution quality, spreads, slippage, latency, broker-specific costs, and paper-trading validation remain open issues.

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
9. Compare ML-filtered strategies against no-filter and random baselines.
10. Run hostile robustness checks such as entry delay, exit delay, higher costs, and monthly breakdowns.

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
- selected crypto pairs for related research
- high-frequency OHLC-style time series
- parquet-based storage for faster loading and analysis

Raw data is **not included** in this repository.

The data pipeline is designed around reproducible local processing:

```text
raw data → cleaned time series → parquet files → feature generation → research outputs
