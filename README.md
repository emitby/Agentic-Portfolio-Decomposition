# Agentic Portfolio Decomposition

Explain a fund's daily returns as a weighted mix of simpler building blocks
(ETFs), then test how well that mix predicts the fund out of sample.

The notebook currently uses **Fidelity Contrafund (FCNTX)** as the target and
six ETFs as the components:

| Ticker | Exposure |
|--------|----------|
| SPY | US large-cap equities |
| QQQ | Nasdaq-100 (large-cap growth / tech) |
| TLT | 20+ year US Treasuries |
| IEF | 7–10 year US Treasuries |
| GLD | Gold |
| USO | Crude oil |

## What the notebook does

[`Agentic-Portfolio-Decomposition.ipynb`](Agentic-Portfolio-Decomposition.ipynb) runs top to bottom in these steps:

1. **Choose tickers.** `target_ticker` and `component_tickers` are set in one
   cell. An interactive `get_tickers()` prompt is included but commented out
   for later use.
2. **Download prices.** Ten years of daily closing prices from Yahoo Finance
   (via `yfinance`), adjusted for dividends and splits so they reflect total
   return.
3. **Clean the data.** Sort dates, drop duplicate and empty dates, and fill any
   missing price forward from the last known value.
4. **Compute daily returns** and check them: compounding the daily returns
   must reproduce each ticker's price change over the same dates (differences
   are around 1e-14, i.e. rounding only).
5. **Walk-forward regression.** Using `lookback = 120` and `projection = 30`
   trading days, repeatedly:
   - fit an ordinary least-squares regression (with intercept) of the target's
     returns on the components' returns over 120 days,
   - apply those weights to the next 30 days of component returns,
   - record the correlation of predicted vs actual target returns, plus the
     prediction and actual return for the first day after the 120.

   The window moves forward one day at a time, and the weights only use data
   from before the days they predict. Two charts show the rolling correlation
   and the growth of $1 from the 1-day predictions vs the actual fund.
6. **80/20 train/test regression.** Fit once on the first 80% of the history,
   predict the last 20% with those fixed weights, and chart growth of $1 for
   predicted vs actual, with the correlation of the two lines.

## Results so far

From a run on 2026-11-03 (results change as new data arrives):

- **Walk-forward:** mean 30-day correlation of **0.958** across 2,363 windows;
  the growth-of-$1 lines correlate at **0.998**. The fit weakens from 2024
  onward, with a sharp dip to about 0.63 in spring 2025.
- **80/20 split:** trained 2016-11 to 2024-10, tested 2024-10 to 2026-11. The
  growth-of-$1 lines correlate at **0.988**. Fitted weights: QQQ 0.58,
  SPY 0.37, IEF −0.18, TLT 0.05, GLD 0.03, USO 0.01 — roughly 95% equity
  with a growth tilt.

## Setup

Requires Python 3.12.

```bash
python -m venv .venv
# Windows: .venv\Scripts\activate    macOS/Linux: source .venv/bin/activate
pip install -r requirements.txt
```

Open the notebook in VS Code or Jupyter, select the `.venv` kernel, and run all
cells. Downloading prices needs an internet connection.

Optional: create a `.env` file containing `MPLCONFIGDIR=.cache/matplotlib` so
matplotlib keeps its cache inside the project (VS Code passes `.env` to the
notebook kernel).

**Note:** FCNTX is a mutual fund, so its price for the current day is published
only after the market closes. If you run the notebook during the trading day,
that day's FCNTX price is filled from the previous day and its return shows as 0%.

## Next steps

Linear regression is the baseline. The next phase adds an **agentic workflow**:
AI agents working within a structured scaffold of tools, data access and
evaluation checks to build more sophisticated predictions than a single linear
model. Ideas being explored include:

- Agents that propose and test alternative models (for example regularized or
  non-linear regressions, regime-dependent weights, or different lookback
  lengths) and compare them on the same out-of-sample tests.
- Agents that choose or expand the set of component tickers.
- A consistent evaluation harness so every candidate model is scored the same
  way against the linear-regression baseline, with no look-ahead.

## Project structure

```
Agentic-Portfolio-Decomposition.ipynb   the analysis notebook
requirements.txt                        pinned Python dependencies
tasks/todo.md                           task log
CLAUDE.md                               instructions for the Claude Code assistant
```
