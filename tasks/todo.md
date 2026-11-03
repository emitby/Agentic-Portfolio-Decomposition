# Todo

## Project setup
- [x] Read CLAUDE.md
- [x] Create Jupyter notebook `Agentic-Portfolio-Decomposition.ipynb`
- [x] Add `.gitignore`
- [x] Initialize git repo and make initial commit
- [x] Create public GitHub repo `Agentic-Portfolio-Decomposition` and push

## Review
- Project scaffolded with notebook, `.gitignore`, and task tracking.

## Live notebook refresh
- [x] Add `.vscode/settings.json` with `files.autoSave: afterDelay` (1s) so the open notebook is never dirty and VS Code reloads Claude's on-disk edits automatically

## Ticker input
- [x] Add `get_tickers()` to the notebook: prompts for target + comma-separated components, normalizes, de-duplicates, drops target from components, re-prompts on empty input
- [x] Add cell storing results in `target_ticker` / `component_tickers`
- [x] Verified with simulated input (blank, mixed case, duplicates, target in list)
- [x] Commented out both ticker cells (to be re-enabled later)
- [x] Hard-coded target_ticker='SPY' and component_tickers in the empty cell below

## Price data
- [x] Install yfinance into .venv; cache pointed at .cache/yfinance
- [x] `get_close_prices()` downloads 10y daily adjusted closes (target + components, de-duplicated)
- [x] `clean_prices()` sorts, drops dup/empty dates, forward-fills, drops incomplete leading rows
- [x] Verified: 2513 days x 6 tickers, 0 NaN; synthetic gap test fills forward correctly; yfinance pandas warnings suppressed

## Daily returns
- [x] `to_daily_returns()` = pct_change, first (NaN) row dropped
- [x] `check_returns()` compares prod(1+r)-1 with P_last/P_first-1 per ticker
- [x] Verified: all deltas ~1e-14 (floating-point noise), 2512 return days, 0 NaN
- [x] Set lookback = 120, projection = 30 in a new cell after the returns check

## Rolling regression
- [x] Target changed SPY -> FCNTX (SPY as both target and component made the fit trivial)
- [x] Install matplotlib; `.env` sets MPLCONFIGDIR=.cache/matplotlib so its cache stays in the project
- [x] `rolling_regression()`: OLS with intercept on `lookback` rows, predict next `projection` rows, record correlation + 1-day predicted/actual; step 1 day; full windows only
- [x] Charts: rolling correlation; growth of $1 predicted vs actual with stream correlation
- [x] Verified: whole notebook re-run; one window cross-checked with normal-equations solve (diffs ~1e-16); 2363 windows, mean corr 0.958, stream corr 0.9983

## 80/20 split regression
- [x] Fit OLS (with intercept) on first 80% of returns, predict last 20% with fixed weights
- [x] Chart growth of $1 predicted vs actual on test period with stream correlation
- [x] Verified: train 2016-11-04..2024-10-30 (2009 d), test 2024-10-31..2026-11-03 (503 d), no overlap; stream corr 0.988
- [x] Added requirements.txt (pinned direct deps)
- [x] Added README.md (overview, steps, results, setup, next steps: agentic workflow)
