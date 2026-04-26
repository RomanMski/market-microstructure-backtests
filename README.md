# market-microstructure-backtests

High-frequency XAUUSD market data research project focused on short-horizon mean-reversion, transaction costs, and robust backtesting.

The project studies how gold behaves after short-term price deviations from rolling moving averages. The goal is not to present a live trading system, but to build a serious research workflow: data preparation, signal discovery, sequential backtesting, cost sensitivity, walk-forward validation, event filtering, and robustness checks.

> **Note:** This is a research project, not a live trading system. Historical backtest results are not live trading claims. Execution quality, spreads, slippage, latency, broker-specific costs, and paper-trading validation remain open issues.

---

## What This Project Is About

I started with a simple question:

> When price moves unusually far below a rolling moving average, does it tend to rebound over the next few minutes?

To test this, I built a research pipeline around 1-second XAUUSD data.

The project evolved from basic signal exploration into a more complete backtesting workflow:

1. Load and clean high-frequency market data.
2. Store the data efficiently with parquet.
3. Compute rolling moving averages and deviation signals.
4. Measure how price behaves after different deviation levels.
5. Build sequential backtests with realistic trade rules.
6. Add transaction costs and stress-test assumptions.
7. Use walk-forward train/validation/test splits.
8. Compare filtered signals against unfiltered and random baselines.
9. Run hostile checks such as entry delay, exit delay, fixed exits, flexible exits, and monthly breakdowns.

The main idea is simple:

> A signal is only interesting if it still looks reasonable after costs, delays, and robustness checks.

---

## Selected Figures

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
- high-frequency OHLC-style time series
- parquet-based storage for faster loading and analysis

Raw data is **not included** in this repository.

The local workflow is roughly:

```text
raw data -> cleaned time series -> parquet files -> feature generation -> research outputs
```

---

## Signal Research

The first part of the project studies price deviation from rolling moving averages.

Examples:

- MA300
- MA500
- percentage distance from the moving average
- future maximum rebound over a fixed horizon
- future adverse movement over the same horizon

A typical research question is:

> If price is 0.10% below MA300, what is the distribution of the maximum rebound over the next 2,000 seconds?

This first step is not yet a trading strategy. It is just a way to understand whether the market has a measurable conditional response after short-term deviations.

---

## Sequential Backtesting

After the signal research, I converted the idea into sequential trade simulations.

The backtest includes:

- one open trade at a time
- fixed take-profit
- stop-loss
- maximum holding time
- transaction-cost assumptions
- different exit rules
- train/validation/test separation

Exit types tested include:

- fixed TP/SL
- protected flexible TP
- unprotected flexible TP
- time-based exit

This matters because high-frequency strategies can look good if the backtest is too generous. I wanted to check whether the idea still worked after making the simulation more realistic.

---

## Event Filtering

A later part of the project uses an event-filtering approach.

The basic idea:

1. Generate candidate events where price deviates below the moving average.
2. Label historical events by their future rebound and drawdown.
3. Train a simple model using only information known at entry time.
4. Use the model score to filter for higher-quality candidate events.
5. Compare the filtered strategy against unfiltered and random baselines.

Entry-time features include:

- deviation from MA300 / MA500
- recent short-horizon returns
- moving-average slope
- rolling volatility
- distance from recent highs/lows
- time-of-day features

The goal is not to magically predict the future, but to test whether some deviation events are statistically better than others before the rebound happens.

---

## Walk-Forward Validation

To reduce overfitting, I used time-based splits.

Example long-horizon setup:

- **Train:** 2023
- **Validation:** 2024
- **Final test:** 2025-2026

The model is trained on the training period. Strategy settings are selected on the validation period. The final test period is then used only after the rules are fixed.

This is important because testing and tuning on the same period can easily create misleading results.

---

## Robustness Checks

Several hostile checks are included to see whether the results depend on fragile assumptions.

Examples:

- transaction costs from 1 to 10 bps
- fixed TP only
- protected flexible TP
- unprotected flexible TP
- entry delay
- exit delay
- extra slippage assumptions
- no-filter baseline
- random same-count baseline
- monthly performance breakdown

The point of these tests is to attack the strategy from different angles.

If a result only works under perfect assumptions, it is probably not useful. If it still works after harsher assumptions, it becomes more interesting as a research finding.

---

## What I Learned

The first version of the signal produced many small mean-reversion trades. That looked interesting, but many small trades are very sensitive to costs.

This shifted the project toward:

- fewer but higher-quality candidate events
- cost-aware filtering
- walk-forward validation
- comparing against baselines
- checking whether results survive delays and stricter exit assumptions

The most useful lesson was that a signal can be statistically visible but still not economically useful unless it survives costs and execution assumptions.

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
|
├── README.md
├── .gitignore
|
├── notebooks/
|   └── cleaned research notebooks
|
├── src/
|   └── reusable Python modules
|
└── reports/
    └── figures/
        └── selected research figures
```

The repository is currently being cleaned and organized. Raw data, large parquet files, and local output files are intentionally excluded.

---

## Limitations

This project is research-only and not a live trading system.

Important remaining limitations include:

- historical results may not survive live execution
- broker-specific spreads and commissions are not fully modelled
- fill quality and latency are simplified
- slippage can change results significantly
- XAUUSD execution depends strongly on the trading venue
- high-frequency results are very sensitive to small cost assumptions
- flexible exit logic may be difficult to implement exactly in live markets
- paper trading and broker-specific execution modelling would be needed before any live deployment

---

## Next Steps

Planned improvements:

- clean notebook outputs for public presentation
- modularize research code into reusable Python files
- add more conservative execution modelling
- add paper-trading style forward tests
- compare performance across different market regimes
- test whether similar structures appear in other liquid instruments
- improve visual reports and summary dashboards

---

## Purpose

This repository is part of my portfolio as a Financial Mathematics student interested in quantitative research, statistical modelling, high-frequency data analysis, and systematic trading research.
