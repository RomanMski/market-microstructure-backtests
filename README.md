# XAUUSD Intraday Mean-Reversion Research

Research archive for a short-horizon mean-reversion test on 1-second XAUUSD data.

The study evaluates whether large negative deviations from rolling moving averages contain enough forward rebound to remain meaningful after transaction costs, execution-delay assumptions, and holding-time constraints.

Related continuation:
[Adaptive Parameter Optimisation Algorithm](https://github.com/RomanMski/Adaptive_Parameter_Optimisation_Algorithm)

> This repository is for research documentation only. Backtest results are not live trading results, trading advice, or evidence of deployability.

## Abstract

The experiment scans XAUUSD price deviations from rolling moving averages, labels forward rebound and drawdown, and converts candidate regions into sequential trade simulations. Early in-sample results produced many small mean-reversion trades, but the edge became highly sensitive once costs and stricter exits were introduced.

The main finding is conditional rather than conclusive: some parameter regions show short-term rebound behaviour, but the setup is fragile under transaction costs and manual parameter selection. This motivated a second version of the work focused on adaptive parameter selection and cross-market testing.

## Research Design

| Area | Specification |
| --- | --- |
| Instrument | XAUUSD |
| Frequency | 1-second data |
| Source | Dukascopy historical data |
| Signal | Negative deviation from rolling moving average |
| Event label | Forward rebound and drawdown after signal time |
| Backtest style | Sequential, one open position at a time |
| Validation | Time-based train / validation / final test split |
| Main risk | Cost sensitivity and parameter overfitting |
| Status | First research version, not a packaged trading system |

## Data Workflow

```text
raw Dukascopy files
-> cleaned time series
-> parquet storage
-> rolling features
-> deviation events
-> sequential backtests
-> diagnostic reports
```

The raw files were converted to parquet to make repeated scans over high-frequency data practical.

## Methodology

### Event Study

For each moving-average window, the study measures how price behaves after large negative deviations.

Example:

```text
price < MA300 by 0.10%
-> measure maximum rebound and drawdown over the next 2,000 seconds
```

The event study is used to identify regions with enough observations and enough forward movement to justify backtesting.

### Sequential Backtest

Candidate regions are tested as trade rules with:

- one open trade at a time
- fixed take profit
- stop loss
- maximum holding time
- transaction cost assumptions
- delayed entry and delayed exit checks
- fixed and flexible exit variants

The purpose is to test whether the raw event behaviour survives a less generous simulation.

### Event Filtering

A later stage adds a simple event filter trained only on information available at entry time.

Features include:

- deviation from MA300 / MA500
- short-horizon returns
- moving-average slope
- rolling volatility
- distance from recent highs and lows
- time-of-day variables

The filter is used as a diagnostic for event quality, not as a claim of stable prediction.

## Validation

The main split is time based:

| Period | Use |
| --- | --- |
| 2023 | Training |
| 2024 | Validation and parameter selection |
| 2025-2026 | Final test |

Robustness checks include:

- transaction-cost sensitivity
- additional slippage
- entry and exit delay
- fixed-exit variants
- flexible-exit variants
- unfiltered baseline
- random same-count baseline
- monthly performance breakdown

## Results Interpretation

The strongest lesson from this version is that the signal is cost-sensitive.

Small mean-reversion trades can look attractive before costs, but many disappear after fees, slippage, and tighter exits. More interesting regions are those with enough trade count, enough entry-to-exit distance, and less dependence on a single parameter choice.

The project should be read as a first research pass. It identifies useful diagnostics and failure modes, but it does not establish a deployable strategy.

## Diagnostics

| Output | Description |
| --- | --- |
| <img src="reports/figures/xauusd_rebound_vs_entrydev_2x2_1y.png" alt="Entry deviation versus future rebound" width="430"> | Event-study view of entry deviation versus forward rebound. Used to inspect whether candidate areas are broad or isolated. |
| <img src="reports/figures/xauusd_HOSTILE_AUDIT_harsh_case_equity.png" alt="Hostile audit equity curve" width="430"> | Equity curve under harsher assumptions. Used as a stress check against overly generous execution assumptions. |
| <img src="reports/figures/xauusd_strict_feature_importance.png" alt="Feature importance for event filtering" width="430"> | Feature-importance diagnostic for the event filter. Used to inspect which entry-time variables the model relied on. |
| <img src="reports/figures/xauusd_LONGHORIZON_test_cost_sensitivity.png" alt="Cost sensitivity analysis" width="430"> | Cost-sensitivity test. Important because short-horizon strategies can fail quickly as frictions increase. |

## Repository Status

```text
reports/figures/   saved figures from research runs
notebooks/         placeholder for public notebooks
src/               placeholder for public research code
```

This repository currently documents the research and selected outputs. It is not yet a clean reproduction package.

## Limitations

- Single-instrument study.
- No order book data.
- No live or paper-trading validation.
- Simplified spread, latency, and fill assumptions.
- Manual parameter selection still introduces data-mining risk.
- Public repository does not yet include the full research pipeline.

## Tools

Python, pandas, polars, NumPy, scikit-learn, matplotlib, Jupyter, parquet, R.

## Next Version

The next version moves away from a fixed single-market setup and tests a more adaptive workflow across multiple instruments and regimes:

[Adaptive Parameter Optimisation Algorithm](https://github.com/RomanMski/Adaptive_Parameter_Optimisation_Algorithm)
