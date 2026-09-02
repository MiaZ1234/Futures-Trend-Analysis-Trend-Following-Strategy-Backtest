# Multi-Market Trend-Following Strategy with Walk-Forward Optimization

A quantitative research project evaluating the out-of-sample performance and robustness of a channel-breakout trend-following strategy across futures markets.

The project combines **statistical price diagnostics, transaction-cost-aware backtesting, exhaustive parameter optimization, rolling walk-forward validation, sensitivity analysis, cross-market replication, and robustness diagnostics**.

The research focuses on **CBOT Soybeans (SY)** as the primary market and **SHFE Gold Futures (AUG)** as the secondary-market replication.

---

## Project Overview

The objective of this project is to investigate whether a systematic channel-breakout strategy can identify persistent futures price trends and retain its performance outside the sample used for parameter estimation.

The analysis addresses four main questions:

1. Do futures prices exhibit statistically detectable departures from random-walk behavior?
2. Can a channel-breakout strategy exploit longer-horizon price persistence after transaction costs?
3. How much strategy performance survives under strict rolling out-of-sample evaluation?
4. Are the results robust across markets, estimation windows, and alternative diagnostic assumptions?

The baseline validation framework uses a **4-year in-sample / 3-month out-of-sample rolling design**, with parameters re-optimized before each OOS period.

---

## Markets and Data

| Role | Market | Exchange | Frequency | Available History |
|---|---|---|---|---|
| Primary | CBOT Soybeans (SY) | CME / CBOT | 5-minute OHLC | Jul 1982 – Apr 2026 |
| Secondary | SHFE Gold Futures (AUG) | SHFE | 5-minute OHLC | May 2018 – Apr 2026 |

The finalized datasets contain approximately:

- **495,740 SY observations**
- **138,744 AUG observations**

Data preprocessing includes timestamp validation, duplicate detection, missing-value checks, OHLC consistency checks, price validation, and trading-session diagnostics.

---

## Research Pipeline

The project follows a sequential research workflow:

```text
Market Analysis
      ↓
Data Validation & Preprocessing
      ↓
Random-Walk / Price-Dynamics Tests
      ↓
Strategy Engine Implementation
      ↓
Exhaustive Parameter Optimization
      ↓
Rolling Walk-Forward OOS Testing
      ↓
Full-Sample Hindsight Benchmark
      ↓
Window Sensitivity Analysis
      ↓
Cross-Market Replication
      ↓
Robustness & Limitation Diagnostics
```

---

## 1. Statistical Price Diagnostics

Before implementing the trading strategy, the project examines whether the underlying price series exhibit departures from random-walk behavior.

Two approaches are used:

### Variance Ratio Test

The analysis implements the **Lo–MacKinlay heteroskedasticity-robust Variance Ratio test** across multiple horizons.

For SY, statistically significant variance ratios below one are observed at several short-to-intermediate horizons, with the strongest evidence around approximately **20–80 minutes**.

This is consistent with short-horizon mean reversion.

### Push-Response Analysis

Positive and negative price shocks are analyzed separately to examine the subsequent directional response.

The results indicate different short-horizon dynamics across the two markets:

- SY exhibits stronger reversal behavior, particularly following positive shocks.
- AUG displays more continuation-like behavior.

These short-horizon diagnostics do not necessarily contradict longer-horizon trend following because the trading strategy operates over substantially longer channel horizons.

---

## 2. Strategy Engine

The trading system is based on a **Channel WithDDControl** trend-following framework.

The Python implementation includes:

- prior-bar rolling high/low breakout channels;
- long and short positions;
- channel reversals;
- trailing drawdown-control stops;
- bar-level position tracking;
- explicit futures point-value conversion;
- transaction-cost/slippage accounting;
- trade-level P&L attribution;
- cumulative equity and drawdown tracking.

Current-bar information is excluded from the channel used to generate the corresponding breakout signal, preventing look-ahead bias in channel construction.

The strategy implementation is validated through accounting reconciliation and consistency checks between optimization and path-based backtesting engines.

---

## 3. Exhaustive Parameter Optimization

Two strategy parameters are optimized:

| Parameter | Range | Step |
|---|---:|---:|
| `ChnLen` | 500 – 10,000 bars | 10 |
| `StpPct` | 0.005 – 0.100 | 0.001 |

This produces:

**951 × 96 = 91,296 parameter combinations per optimization window.**

The optimization objective is:

> **Net Profit / |Maximum Drawdown|**

Rather than using random search or an approximate optimization procedure, the complete prescribed parameter grid is evaluated for every optimization window.

The computational implementation uses optimized rolling-channel construction and parallelized stop-parameter evaluation to make repeated full-grid optimization feasible.

---

## 4. Rolling Walk-Forward Validation

The baseline walk-forward framework uses:

- **4 years in-sample**
- **3 months out-of-sample**
- quarterly re-optimization
- **91,296 parameter combinations per IS window**

For every OOS quarter:

1. strategy parameters are selected using only the preceding in-sample window;
2. the selected parameters are carried into the following OOS period;
3. only OOS P&L is retained for performance evaluation;
4. quarterly OOS P&L is stitched chronologically into a continuous OOS equity curve.

The main analysis uses an **IS-conditioned OOS implementation**, allowing the strategy state entering each OOS period to be generated from its corresponding historical in-sample path while preventing parameter information from future periods from entering the optimization.

---

## Rolling Out-of-Sample Results

| Metric | CBOT Soybeans (SY) | SHFE Gold (AUG) |
|---|---:|---:|
| Initial Equity | 100,000 | 100,000 |
| Ending Equity | 372,260.74 | 955,221.26 |
| Net Profit | 272,260.74 | 855,221.26 |
| CAGR | **3.36%** | **82.58%** |
| Maximum Drawdown | **-6.79%** | **-9.91%** |
| Daily Sharpe Ratio | **1.345** | **3.609** |
| Calmar Ratio | **0.495** | **8.336** |

Because SY and AUG cover different historical periods and have different contract economics and market structures, these figures are presented as a **cross-market robustness comparison rather than a direct performance ranking**.

![Cross-Market Rolling OOS Performance](Output/figures/cross_market_oos_comparison.png)

---

## 5. Primary Market — CBOT Soybeans

The finalized SY walk-forward experiment contains:

- **159 rolling OOS quarters**
- approximately **449,885 OOS bars**
- **2,101 completed OOS trades**
- **108 positive quarters**
- **44 negative quarters**
- **7 flat quarters**

### OOS Performance

| Metric | Result |
|---|---:|
| CAGR | **3.36%** |
| Daily Sharpe | **1.345** |
| Maximum Drawdown | **-6.79%** |
| Calmar | **0.495** |
| Profit Factor | **1.974** |
| Trade Win Rate | **43.65%** |

The SY strategy retains a positive historical edge after repeated re-estimation, transaction costs, and strict rolling OOS evaluation.

The result is consistent with historical generalization within the available dataset, but it is not interpreted as proof that the strategy is free from overfitting.

---

## 6. Full-Sample Hindsight Benchmark

A separate optimization is performed using the entire available history of each market.

The full-sample optimization serves as a **hindsight in-sample benchmark**.

It is not treated as a tradable OOS strategy.

For SY, the full-history optimum is:

- `ChnLen = 3810`
- `StpPct = 0.009`

with:

- CAGR: **2.28%**
- Maximum Drawdown: **-3.68%**
- Daily Sharpe: **1.191**
- Calmar: **0.621**

Comparing the rolling OOS experiment with this benchmark provides a way to examine whether the apparent strategy edge collapses once parameter selection is restricted to historically available information.

The OOS results remain positive, although the comparison is interpreted using normalized performance and trade-quality measures rather than raw cumulative P&L because the evaluation periods differ.

---

## 7. Walk-Forward Sensitivity Analysis

The baseline 4-year / 3-month configuration is not treated as an ex-post selected optimum.

Alternative combinations of in-sample length \(T\) and OOS horizon \(\tau\) are evaluated as robustness checks.

For SY, common-period results across the tested specifications remain relatively stable:

- CAGR: approximately **3.64% – 3.67%**
- Daily Sharpe: approximately **1.34 – 1.45**
- Maximum Drawdown: approximately **-9.18% to -7.04%**
- Calmar: approximately **0.40 – 0.52**

The results suggest that the historical SY edge is not uniquely dependent on the baseline window configuration, although risk measures remain sensitive to the precise specification.

---

## 8. Secondary-Market Replication — SHFE Gold Futures

The complete research framework is independently replicated on **SHFE Gold Futures (AUG)** using:

- the same strategy logic;
- the same optimization objective;
- the same 91,296-point parameter grid;
- the same baseline 4-year IS / 3-month OOS design.

The AUG rolling experiment contains:

- **15 OOS quarters**
- approximately **65,376 OOS bars**
- **173 completed OOS trades**
- **15 / 15 positive OOS quarters**

### OOS Performance

| Metric | Result |
|---|---:|
| CAGR | **82.58%** |
| Daily Sharpe | **3.609** |
| Maximum Drawdown | **-9.91%** |
| Calmar | **8.336** |
| Profit Factor | **7.186** |
| Trade Win Rate | **58.38%** |

The magnitude of these results is substantially stronger than the primary-market result.

Rather than interpreting the headline performance at face value, the project therefore performs additional concentration, parameter-boundary, and data-path diagnostics.

---

## 9. AUG Robustness Diagnostics

### Temporal Concentration

AUG OOS profitability is meaningfully concentrated in a relatively small number of quarters:

- largest quarter: **41.06%** of total net OOS P&L;
- top three quarters: **63.11%** of total net OOS P&L.

This indicates that the headline result is not evenly distributed through time.

### Session-Gap Attribution

Approximately **76.67%** of AUG net OOS P&L is associated with session-gap bars.

Only approximately **23.33%** is attributed to non-gap bars.

A diagnostic excluding session-gap P&L attribution remains profitable, but performance is materially weaker:

| Metric | Baseline OOS | Non-Gap Diagnostic |
|---|---:|---:|
| CAGR | **82.58%** | **34.00%** |
| Calmar | **8.34** | **2.70** |
| Maximum Drawdown | **-9.91%** | **-12.59%** |

This is an important limitation because the 5-minute dataset does not observe the complete price path occurring inside inter-session gaps.

### Parameter Boundary

The optimized AUG stop parameter repeatedly selects:

`StpPct = 0.005`

which is the lower boundary of the prescribed parameter grid.

The lower-bound selection frequency is **100%** in both the baseline rolling optimization and the tested AUG sensitivity specifications.

This indicates that the stop parameter is not fully identified within the prescribed optimization range and should be interpreted as a model limitation rather than evidence of parameter stability.

![AUG Robustness Diagnostics](Output/figures/aug_robustness_diagnostics.png)

---

## 10. AUG Window Sensitivity

Because the available AUG history is substantially shorter than the SY history, the feasible sensitivity design uses:

- \(T \in \{4,5,6\}\) years;
- \(\tau \in \{3,6\}\) months.

Across all six tested specifications, the strategy remains profitable on the identical common OOS comparison period.

Common-period results span approximately:

| Metric | Range |
|---|---:|
| CAGR | **167.68% – 183.05%** |
| Daily Sharpe | **3.98 – 4.30** |
| Maximum Drawdown | **-14.14%** |
| Calmar | **11.86 – 12.94** |

The results show limited sensitivity to the tested window specifications.

However, the short available history and the small number of OOS windows—particularly for the longest estimation windows—limit the strength of the statistical inference.

---

## Key Findings

1. **Price dynamics are horizon-dependent.**  
   Short-horizon mean reversion can coexist with profitable longer-horizon breakout behavior.

2. **SY retains a positive historical edge under rolling OOS evaluation.**  
   The result survives repeated parameter re-estimation and prescribed transaction costs, although performance remains moderate.

3. **The framework transfers across markets.**  
   Applying the same research and optimization framework to AUG produces strong historical rolling OOS results.

4. **Strong backtests require additional diagnostics.**  
   AUG's headline performance is materially affected by temporal concentration, session-gap attribution, and persistent lower-bound stop selection.

5. **Full-sample optimization is treated only as a hindsight benchmark.**  
   It is not presented as a competing OOS strategy.

6. **Robustness is evaluated beyond the equity curve.**  
   The project examines parameter sensitivity, trade-level accounting, performance concentration, market replication, and data-path dependence.

---

## Repository Structure

```text
.
├── README.md
├── requirements.txt
├── .gitignore
│
├── data/
│
├── notebooks/
│   ├── 01_market_analysis.ipynb
│   ├── 02_data_preprocessing.ipynb
│   ├── 03_random_walk_tests.ipynb
│   ├── 04_strategy_engine.ipynb
│   ├── 05_sy_walk_forward_optimization.ipynb
│   ├── 06_sy_full_sample_optimization.ipynb
│   ├── 07_sy_oos_vs_insample.ipynb
│   ├── 08_sy_window_sensitivity.ipynb
│   ├── 09_aug_walk_forward_optimization.ipynb
│   ├── 10_aug_full_sample_optimization.ipynb
│   ├── 11_aug_window_sensitivity.ipynb
│   ├── 12_aug_robustness_assessment.ipynb
│   ├── 13_aug_procedure_comparison.ipynb
│   └── 14_project_summary_figures.ipynb
│
├── Output/
│   ├── figures/
│   │   ├── cross_market_oos_comparison.png
│   │   └── aug_robustness_diagnostics.png
│   └── result/
│
└── reference/
    ├── main.m
    └── ezread.m
```

The notebooks are organized as a sequential research pipeline:

- **01–04:** shared market analysis, data validation, statistical testing, and strategy implementation;
- **05–08:** primary-market research on CBOT Soybeans;
- **09–13:** secondary-market replication and robustness analysis on SHFE Gold Futures;
- **14:** project-level summary figures.

---

## Reproducibility

The project is implemented primarily in Python.

Core libraries include:

- NumPy
- pandas
- Matplotlib
- SciPy
- Numba
- PyArrow

Install the required dependencies with:

```bash
pip install -r requirements.txt
```

Then launch Jupyter:

```bash
jupyter lab
```

The notebooks are designed to be read in numerical order.

Data preprocessing should be completed before downstream statistical tests and strategy experiments because later notebooks consume the cleaned datasets generated during preprocessing.

Generated analytical outputs are stored under:

```text
Output/result/
```

and project-level figures under:

```text
Output/figures/
```

---

## Data Availability

The project uses 5-minute futures OHLC data.

Depending on the licensing or redistribution restrictions associated with the original source data, raw market data may not be suitable for public redistribution.

The repository therefore separates source data, analysis notebooks, and generated research outputs so that the research methodology can be reviewed independently of data redistribution.

---

## Reference Code

The `reference/` directory contains the original MATLAB implementation used as a strategy-logic reference.

The reference code is retained to document the baseline trading rules and to support validation of the independently implemented Python strategy engine.

The Python implementation, optimized backtesting framework, exhaustive grid search, rolling walk-forward evaluation, sensitivity analysis, cross-market replication, and robustness diagnostics are contained in the research notebooks.

---

## Limitations

The results are historical research results and should **not** be interpreted as expected live-trading performance.

Important limitations include:

- historical parameter optimization remains exposed to data-snooping and model-selection risk;
- transaction costs are represented through the specified slippage assumptions;
- execution within a 5-minute OHLC bar cannot be fully reconstructed;
- inter-session price paths are not observed in the AUG dataset;
- AUG has a substantially shorter history than SY;
- AUG OOS performance is materially concentrated in a small number of periods;
- a substantial share of AUG P&L is associated with session-gap bars;
- the AUG stop parameter repeatedly reaches the lower optimization boundary;
- futures market structure and contract economics differ across the two markets.

These limitations are retained explicitly because evaluating the credibility of a backtest is a central part of the research process.

---

## Project Background

The initial strategy specification was motivated by coursework in financial price analysis.

The project was subsequently **independently restructured, reimplemented, validated, and extended** into the research workflow presented in this repository, including data-quality auditing, Python strategy implementation, exhaustive optimization, rolling out-of-sample evaluation, sensitivity testing, secondary-market replication, and additional robustness diagnostics.

The repository is maintained as a personal quantitative research project for portfolio and technical-review purposes.
