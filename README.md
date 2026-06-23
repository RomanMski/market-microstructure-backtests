# XAUUSD Microstructure Mean-Reversion Study

Short-horizon research note on 1-second XAUUSD data.

The project tests whether large negative deviations from rolling moving averages are followed by tradable rebounds after adding transaction costs, holding-time limits, and basic robustness checks.

This is the first version of the research. A more adaptive cross-market version is here:

[Adaptive Parameter Optimisation Algorithm](https://github.com/RomanMski/Adaptive_Parameter_Optimisation_Algorithm)

> Research only. These are historical backtests, not live trading results or trading advice. Execution quality, spreads, slippage, latency, broker-specific costs, and paper/live validation remain outside the scope of this repository.

## Scope

| Item | Description |
| --- | --- |
| Instrument | XAUUSD |
| Frequency | 1-second market data |
| Data source | Dukascopy historical data |
| Signal type | Negative deviation from rolling moving average |
| Main question | Does price rebound enough after extreme short-term deviations to survive costs? |
| Status | First-pass research archive, not a production trading system |

## Research Question

The starting hypothesis was simple:

> When XAUUSD trades far below a slow rolling moving average on a short timeframe, the following price path may contain mean-reversion behaviour.

The useful question is not whether a rebound can be found in-sample. The harder test is whether the effect remains visible after:

- transaction costs
- slippage assumptions
- stop loss and take profit rules
- maximum holding time
- train / validation / test separation
- random and unfiltered baselines

## Data Pipeline

The raw data was converted into a faster research format before running the scans and backtests.

```text
raw Dukascopy files
-> cleaned time series
-> parquet storage
-> rolling features
-> deviation events
-> sequential backtests
-> validation reports
```

Parquet was used because repeated scans over 1-second data are slow in plain CSV form.

## Method

### 1. Deviation scan

For each rolling moving-average window, the research checks how price behaves after large negative deviations from the moving average.

Example event:

```text
price is 0.10% below MA300
-> measure maximum rebound and drawdown over the next 2,000 seconds
```

The scan was used to find regions with enough observations and enough average rebound to justify a sequential backtest.

### 2. Sequential backtest

The event scan was then converted into trade simulations with one open position at a time.

Tested components included:

- fixed take profit
- stop loss
- maximum holding time
- transaction cost assumptions
- delayed entry / delayed exit checks
- flexible exit variants

This step was mainly used to test whether the raw signal survived more realistic trading assumptions.

### 3. Event filtering

A later version of the project added a simple event filter. Candidate events were labelled by their future rebound and drawdown, then scored using only information available at entry time.

Entry-time features included:

- deviation from MA300 / MA500
- short-horizon returns
- moving-average slope
- rolling volatility
- distance from recent highs and lows
- time-of-day features

The purpose was not to forecast every trade. The purpose was to test whether some deviation events were historically higher quality than others before the rebound occurred.

## Validation

The main validation setup used time-based splits:

| Period | Role |
| --- | --- |
| 2023 | Training |
| 2024 | Validation / parameter selection |
| 2025-2026 | Final test |

The final test period was only used after the rules were fixed. This reduces, but does not remove, data-mining risk.

Additional checks included:

- cost sensitivity from low to harsher assumptions
- fixed-exit and flexible-exit variants
- entry delay and exit delay
- additional slippage
- unfiltered baseline
- random same-count baseline
- monthly performance breakdown

## Main Observations

- The raw deviation signal is very sensitive to transaction costs.
- Many small mean-reversion trades look acceptable before costs and weak after costs.
- Parameter regions with more distance between entry and exit are more interesting than regions that rely on tiny rebounds.
- Maximum holding time matters. Trades that remain open too long often stop behaving like the original short-term setup.
- The project is useful as a first research pass, but the setup is too fixed around one instrument and one manually selected parameter workflow.

These observations motivated the second project, which generalizes the workflow across more markets and uses a more adaptive parameter-selection process.

## Selected Outputs

### Entry Deviation vs Forward Rebound

<p align="center">
  <img src="reports/figures/xauusd_rebound_vs_entrydev_2x2_1y.png" alt="Entry deviation versus future rebound" width="900">
</p>

This chart was used to inspect how different deviation levels relate to future rebound behaviour. The goal was to avoid selecting a parameter area from one isolated result.

### Hostile-Audit Equity Curve

<p align="center">
  <img src="reports/figures/xauusd_HOSTILE_AUDIT_harsh_case_equity.png" alt="Hostile audit equity curve" width="900">
</p>

This run applies harsher assumptions to check whether the backtest depends on overly generous execution settings.

### Feature Importance

<p align="center">
  <img src="reports/figures/xauusd_strict_feature_importance.png" alt="Feature importance for event filtering" width="900">
</p>

The feature-importance plot was used as a diagnostic for the event filter. It shows which entry-time variables were used most by the model.

### Cost Sensitivity

<p align="center">
  <img src="reports/figures/xauusd_LONGHORIZON_test_cost_sensitivity.png" alt="Cost sensitivity analysis" width="900">
</p>

Cost sensitivity is one of the main checks in this project. Short-horizon strategies can fail quickly when fees and slippage are increased.

## Repository Layout

```text
reports/figures/   saved figures from the research runs
notebooks/         reserved for public notebooks
src/               reserved for public research code
```

At the moment this repository is mainly a public research summary with saved outputs. The full code and data pipeline are not packaged as a clean reproduction workflow yet.

## Tools

- Python
- pandas
- polars
- NumPy
- scikit-learn
- matplotlib
- Jupyter
- parquet
- R

## Limitations

- Single-instrument study.
- No order book data.
- No live or paper-trading validation.
- Broker-specific spread, latency, and fill assumptions are simplified.
- Parameter search still carries data-mining risk.
- Public repository does not yet contain a full reproducible code pipeline.

## Next Step

The next version should reduce manual parameter selection and test whether the same idea behaves consistently across different instruments and regimes.

That continuation is developed here:

[Adaptive Parameter Optimisation Algorithm](https://github.com/RomanMski/Adaptive_Parameter_Optimisation_Algorithm)
