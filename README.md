# Gap-Stability Alpha

An intraday mean-reversion strategy that trades overnight gaps on NSE (Nifty 100) stocks, filtered by previous-day entropy stability and Nifty 50 regime alignment.

## Strategy Overview

| Component | Description |
|-----------|-------------|
| **Signal** | α = −(Open − Prev Close) / ATR₁₄,ₜ₋₁ |
| **Direction** | Long if gap-down (α > 0), Short if gap-up (α < 0) |
| **Entry** | 09:20 IST bar open |
| **Exit** | First of: Gap fill · 2.5×ATR stop-loss · Time exit (15:00) |
| **Universe** | 103 Nifty 100 constituent stocks |
| **Capital** | ₹1,00,000 · Max 3 trades/day · Equal-weight |

### Filters (all lookahead-free)

1. **Signal strength** — |Gap / ATR<sub>t-1</sub>| > 0.5  
2. **Entropy filter** — Previous day's normalised Shannon entropy < 0.50 (structured regime)  
3. **Market-wide gap skip** — |Nifty gap / ATR<sub>t-1</sub>| < 1.5 (avoid macro events)  

### Ranking

Trades are ranked daily by a composite score and the top 3 are selected:  
`score = |α| × (1 − entropy) × nifty_alignment_boost`

- Nifty-trend-aligned trades get 1.5× boost  
- Counter-trend trades get 0.7× penalty  

## Lookahead-Free Design

All features used at the point of decision (09:20 market open) are strictly causal:

| Feature | Available at open? | Implementation |
|---------|-------------------|----------------|
| ATR | ✅ Uses ATR<sub>t-1</sub> | `rolling(14).mean().shift(1)` |
| Gap | ✅ Open<sub>t</sub> − Close<sub>t-1</sub> | Both known at open |
| Entropy | ✅ Uses prev day | `entropy.shift(1)` |
| Nifty trend | ✅ Uses prev day | `trend.shift(1)` |
| Nifty gap/ATR | ✅ Gap known, ATR<sub>t-1</sub> | ATR shifted by 1 |

## Results (2020-01 → 2026-01, 15:00 exit)

| Metric | Value |
|--------|-------|
| Total Trades | 589 |
| Annualised Sharpe | 2.12 |
| CAGR | 20.83% |
| Max Drawdown | -7.77% |
| Win Rate | 46.0% |
| Profit Factor | 2.02 |
| Calmar Ratio | 2.68 |
| PSR (vs 0) | 100% ✓ |

> Actual numbers may vary slightly by run — re-execute the notebook to reproduce.

## Project Structure

```
gap_stability_alpha/
├── gap_stability_alpha_backtester.ipynb   # Main notebook (run top-to-bottom)
├── stocks_data/                           # 1-min OHLCV CSVs (NOT in repo – see Data below)
│   ├── RELIANCE_minute.csv
│   ├── HDFCBANK_minute.csv
│   ├── NIFTY 50_minute.csv
│   └── ...
├── requirements.txt
├── README.md
└── .gitignore
```

## Quickstart

```bash
# 1. Clone
git clone <repo-url>
cd gap_stability_alpha

# 2. Create virtual environment
python3 -m venv .venv
source .venv/bin/activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Run the notebook
jupyter notebook gap_stability_alpha_backtester.ipynb
# Execute all cells top-to-bottom. Runtime ≈ 3 minutes.
```

## Data

The `stocks_data/` directory (~4.8 GB) is **not included** in the repository.  
You need to supply your own 1-minute OHLCV data for Nifty 100 stocks and the two indices (`NIFTY 50`, `NIFTY BANK`).

Place the CSVs in `stocks_data/` with the naming convention `<SYMBOL>_minute.csv`.  
Each CSV must have columns: `date, open, high, low, close, volume`  
Timestamps should be IST (UTC+05:30), 1-minute bars.

## Outputs (generated on run)

- `gap_stability_alpha_trade_log.csv` — Full trade-level detail  
- `equity_curve.png` — Portfolio equity, drawdown, per-trade PnL  
- `exit_time_comparison.png` — Multi-exit analysis  
- `monthly_returns_heatmap.png` — Year × Month return heatmap  
- `return_analysis.png` — Distribution & Q-Q plot  
- `advanced_analysis.png` — Direction/exit-type breakdown, rolling Sharpe  
- `gap_entropy_analysis.png` — Signal landscape & bucket analysis  

## License

This project is for educational and research purposes only. Not financial advice.
