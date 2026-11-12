# XAUUSD D1 Quantitative Research: N-Method Backtest & Market Regime Analysis

A reproducible Python research project that backtests three rule-based N-Method strategy variants on XAUUSD daily data, builds a simple trend/regime baseline, and evaluates strategy performance conditional on the detected regime.

> **Research project, not investment advice.** Results are historical simulations, not live trading results or a promise of future performance. The strategy rules are an operational interpretation and are not verified as an official/proprietary Traders Family implementation.

## Research questions

1. Can a simple and interpretable model classify XAUUSD D1 into Bullish, Bearish, and Sideways regimes?
2. How do Bullish N, Bearish N, and Ranging N perform within each detected regime?
3. Does a simple matched-regime filter improve the original unfiltered N-Method results?

## Dataset

- Instrument: XAUUSD (Gold vs US Dollar)
- Timeframe: 1 day (D1)
- Sample used in the current analysis: **1 February 2010 – 11 November 2026**
- Number of daily bars: **5,221**
- Input format: headerless OHLCV CSV (`Date, Open, High, Low, Close, Volume`)

The project bundle contains the local source CSV used to generate the results. It is excluded from Git commits by default; see [`data/README.md`](data/README.md). Before publishing a public repository, confirm the source data's redistribution rights. The notebooks expect the file at `data/XAUUSD_D1.csv`.

## Methodology

### 1. N-Method strategy implementation

The first notebook implements three variants using confirmed swing pivots:

- **Bullish N:** P1 swing low → P2 swing high → P3 higher swing low; buy-stop above P2, stop-loss below P3.
- **Bearish N:** P1 swing high → P2 swing low → P3 lower swing high; sell-stop below P2, stop-loss above P3.
- **Ranging N:** two tests of similar highs and lows; opposing breakout orders are managed with OCO logic.

Default settings include 3 bars on both sides for pivot confirmation, 0.05 ATR entry/stop buffers, 20-bar pending-order expiry, and a fixed 2R take-profit target. If stop-loss and take-profit are both touched in one daily candle, the backtest assumes the stop-loss was hit first. If both sides of an OCO range order trigger in the same candle, the setup is marked ambiguous and excluded from performance statistics.

**Rule-source caveat:** Stop placement is based on a practical interpretation of general chart-pattern guidance, not a verified copy of an official N-Method playbook. See the detailed note in `01_N_Method_Backtest.ipynb` and the [Traders Family chart-pattern article](https://tradersfamily.id/3-jenis-utama-chart-pattern-dalam-trading/).

### 2. Baseline regime classifier

A deliberately simple rule-based model is used as a first benchmark:

| Regime | Classification rule |
|---|---|
| Bullish | ADX(14) ≥ 20 and EMA(50) > EMA(200) |
| Bearish | ADX(14) ≥ 20 and EMA(50) < EMA(200) |
| Sideways | ADX(14) < 20 |
| Unclassified | Indicator warm-up values are unavailable |

EMA alignment defines direction, while ADX estimates trend strength. ADX is not directional by itself. The indicator settings are fixed baseline parameters, not optimized on the full history.

Each N-Method trade is assigned the regime observed on its `signal_date` (the setup-confirmation date), not the entry or exit date. The pending order becomes active on the next daily candle.

## Results

### Regime distribution

| Regime | Daily bars | Share |
|---|---:|---:|
| Bullish | 2,099 | 40.20% |
| Bearish | 1,255 | 24.04% |
| Sideways | 1,771 | 33.92% |
| Unclassified | 96 | 1.84% |
| **Total** | **5,221** | **100.00%** |

### Strategy performance by detected regime

`Net R` is the sum of trade R-multiples. `PF` is profit factor calculated from positive and negative R values. Small subgroups should be interpreted cautiously.

| Strategy | Regime | Trades | Win rate | Net R | PF |
|---|---|---:|---:|---:|---:|
| Bullish N | Bullish | 30 | 33.3% | 0.00 | 1.00 |
| Bullish N | Bearish | 16 | 37.5% | +1.87 | 1.18 |
| Bullish N | Sideways | 26 | 53.8% | **+16.00** | **2.33** |
| Bullish N | Unclassified | 2 | 50.0% | +1.00 | 2.00 |
| Bearish N | Bullish | 20 | 30.0% | -3.08 | 0.78 |
| Bearish N | Bearish | 13 | 30.8% | -1.00 | 0.89 |
| Bearish N | Sideways | 23 | 26.1% | -6.37 | 0.65 |
| Ranging N | Bullish | 4 | 50.0% | +2.00 | 2.00 |
| Ranging N | Bearish | 3 | 33.3% | -1.97 | 0.01 |
| Ranging N | Sideways | 9 | 33.3% | 0.00 | 1.00 |

### Unfiltered baseline vs. matched-regime filter

The matched-regime filter retains Bullish N trades only in Bullish regimes, Bearish N only in Bearish regimes, and Ranging N only in Sideways regimes.

| Strategy | Selection | Trades | Win rate | Net R | PF |
|---|---|---:|---:|---:|---:|
| Bullish N | No regime filter | 74 | 41.9% | **+18.87** | **1.44** |
| Bullish N | Matched regime only (Bullish) | 30 | 33.3% | 0.00 | 1.00 |
| Bearish N | No regime filter | 56 | 28.6% | -10.44 | 0.75 |
| Bearish N | Matched regime only (Bearish) | 13 | 30.8% | -1.00 | 0.89 |
| Ranging N | No regime filter | 16 | 37.5% | +0.03 | 1.00 |
| Ranging N | Matched regime only (Sideways) | 9 | 33.3% | 0.00 | 1.00 |
| **All strategies** | **No regime filter** | **146** | **36.3%** | **+8.46** | **1.09** |
| **All strategies** | **Matched regime only** | **52** | **32.7%** | **-1.00** | **0.97** |

## Key findings

1. **The matched-regime filter did not improve aggregate results in this sample.** Total performance moved from +8.46R (146 trades, PF 1.09) without a filter to -1.00R (52 trades, PF 0.97) with the matched-regime filter.
2. **Bullish N's strongest observed subgroup was Sideways.** It produced +16.00R across 26 trades, with a 53.8% win rate and 2.33 profit factor. This is a useful research hypothesis—not evidence that the setup should be traded only in Sideways conditions.
3. **Bearish N was negative across all three classified regimes** in this sample. It deserves separate review before being included in a combined strategy.
4. **Ranging N was approximately breakeven overall.** Its regime-specific samples are very small, so the observed PF values are unstable.

The counter-intuitive Bullish N / Sideways result may indicate that the setup is confirmed during low directional strength and then captures a subsequent breakout. That explanation is a hypothesis and requires a separate test.

## Limitations and risk controls

- The classifier is intentionally basic and lagging. EMA and ADX can react after a market transition has started.
- The strategy backtest uses daily OHLC bars; intrabar order of events is unknown. Conservative stop-first handling is used when SL and TP are both touched in one bar.
- Spread, commissions, swap, and realistic slippage are not included. Results may deteriorate after transaction costs.
- `OPEN_AT_END` trades are marked to market at the last available close and are not realized closed-trade outcomes.
- Strategy variants are backtested separately. The aggregate table does not enforce shared capital, simultaneous exposure limits, or portfolio-level risk constraints; overlapping trades may overstate deployable performance.
- Some regime/strategy combinations have very few observations.
- All results are in-sample. The analysis must not be presented as out-of-sample validation.
- The proprietary N-Method rulebook has not been independently verified; the code documents its assumptions explicitly.

## Next steps

1. Add chronological train/test splits or walk-forward evaluation.
2. Re-test the Bullish N / Sideways hypothesis on unseen data.
3. Add realistic spread, commission, swap, and slippage assumptions for the intended broker.
4. Evaluate concurrent positions and portfolio-level risk limits.
5. Compare the baseline classifier with more advanced models only after the baseline is validated.

## Project structure

```text
.
├── README.md
├── requirements.txt
├── .gitignore
├── data/
│   ├── README.md
│   └── XAUUSD_D1.csv              # local source data; ignored by Git by default
├── notebooks/
│   ├── 01_N_Method_Backtest.ipynb
│   └── 02_Baseline_Regime_Analysis.ipynb
└── outputs/
    ├── XAUUSD_D1_N_method_trades.csv
    ├── XAUUSD_D1_N_method_summary.csv
    ├── XAUUSD_D1_regime_distribution.csv
    ├── XAUUSD_D1_regime_performance.csv
    ├── XAUUSD_D1_regime_filter_vs_baseline.csv
    └── XAUUSD_D1_N_method_trades_with_regime.csv
```

## Run locally

```bash
# 1. Create and activate an isolated environment
python3 -m venv .venv
source .venv/bin/activate           # macOS / Linux
# .venv\Scripts\activate        # Windows PowerShell

# 2. Install dependencies
python -m pip install --upgrade pip
python -m pip install -r requirements.txt

# 3. Open the notebooks
jupyter lab
```

Run `01_N_Method_Backtest.ipynb` first, then run `02_Baseline_Regime_Analysis.ipynb`. Both notebooks read the source CSV from `data/` and write their outputs to `outputs/`.

## GitHub publishing note

The raw price dataset is excluded from commits by `.gitignore`. Keep it local unless you have explicit permission to redistribute it. The derived CSV outputs and notebooks can be published, but make sure they do not contain any information you are not authorized to share.
