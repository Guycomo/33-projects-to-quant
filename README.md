# 33 Projects to Quant

A self-taught, hands-on roadmap for building quantitative and algorithmic trading skills from scratch — one small project at a time.

## What this is

This repository contains 33 projects, ordered purely by increasing difficulty, that walk from a first "pull one stock's price history" script all the way to a full end-to-end quant research pipeline. Each project stands on its own — the code in any given project runs independently — but the *knowledge* is cumulative: roughly half of what any given project teaches is new, and the other half builds directly on what was learned in the project before it.

Several of the bigger projects are preceded by a **warm-up project** — a small, self-contained piece cut out of the next project's core new idea. The warm-up is meant to be reasoned through independently before the harder project is tackled, so that each big step only ever introduces one genuinely new concept at a time.

The projects are grouped into five phases:

- **Phase 1 — Foundations:** data pipelines, cleaning, and your first backtests
- **Phase 2 — Statistical Strategy Building:** mean reversion, pairs trading, momentum, and a hand-built backtesting engine
- **Phase 3 — Fixed Income, Derivatives and Simulation:** bond math, options pricing, Monte Carlo methods, portfolio optimization
- **Phase 4 — Machine Learning for Trading:** feature engineering, classical ML, time-series models, deep learning, NLP, and reinforcement learning
- **Phase 5 — Systems, Execution and Performance:** market microstructure, execution algorithms, a C++ rewrite, and live paper trading
- **Capstone:** one integrated end-to-end research project

One rule holds across every project starting with #1: guard against lookahead bias, survivorship bias, and overfitting. That discipline is built in from the very first script, not bolted on later.

Below, each project is documented as it's actually built — which may differ in places from the original plan as real implementation choices get made along the way. Think of this README as the honest record of what was built, not just what was planned.

---

## Projects

### Project 1: Single-Ticker Data Fetch and Explore
#### What it does
Pulls one year of daily price history for gold futures (GC=F) using yfinance, then explores the resulting DataFrame: checking its shape and columns, pulling a single column (Close) as a Series, reading off the first and last closing prices, and computing the min, max, and mean close over the period. Total return over the year is calculated directly as the percentage change from the first close to the last. The notebook closes with a matplotlib line chart of the closing price over the full year.
#### What it covers
Fetching real market data from a financial API (yfinance) with no manual downloading
Basic pandas indexing — selecting a single column, using .iloc to grab specific rows
Computing simple descriptive statistics (min, max, mean) directly off a price series
A first hand-rolled return calculation (not a library function) to build intuition for what "return" actually means
Producing a first plot with matplotlib
#### Notes
No database or cleaning logic yet — this project is deliberately just "prove the data pipeline works end to end," nothing more.

## Project 2: Multi-Ticker Data Pipeline with SQLite Storage
### What it does
Extends Project 1 from a single ticker to five: gold futures (GC=F), EUR/USD, GBP/USD, BTC-USD, and Nasdaq futures (NQ=F) — a deliberate mix of asset classes rather than five more stocks. Loops over the list with yfinance, flattening the MultiIndex columns that yf.download returns for some tickers, and writes each row into a local SQLite database (market_data.db) with a prices table keyed on (ticker, date). Uses INSERT OR IGNORE against that composite primary key so re-running the script never creates duplicate rows. Ends by querying row counts per ticker to confirm the fetch worked — with the row counts themselves surfacing a real data quirk: BTC-USD has 366 rows for the year (it trades every day) while the FX and futures tickers have fewer (~252–260, reflecting market holidays and weekends).
### What it covers
Looping over an API call across multiple tickers, with a short time.sleep() between requests to be polite to the data source
Designing a simple SQL schema by hand and creating it with sqlite3 directly (no ORM)
Idempotent writes: the (ticker, date) primary key plus INSERT OR IGNORE means the script can be re-run safely without duplicating data
Handling an inconsistency in the API's return shape (MultiIndex columns) before it silently breaks the insert logic
A first real encounter with the fact that different asset classes don't share the same trading calendar — crypto trades every day, FX and futures don't
### Notes
No cleaning or corporate-actions logic yet - that's Project 3. This one is purely about getting multiple tickers into durable, non-duplicating storage.

## Project 3: Data Cleaning and Corporate Actions Handling
### What it does
Takes the raw daily price data already sitting in the Project 2 SQLite database and puts it through a cleaning pipeline before it's trusted for any later analysis. The pipeline runs in four stages: first, it checks the price history against the official market calendar to flag any trading days that are missing from the data. Second, it adjusts historical prices for stock splits and dividends, so a chart doesn't show a fake "crash" on the day a company did a 2-for-1 split. Third, it scans the adjusted price series for suspicious day-to-day jumps that look like bad data rather than real market moves. Fourth, every flag, fix, and decision made along the way gets written to a cleaning log, so the process is auditable rather than a black box.
### What it covers
Cross-referencing a price series against a real market calendar (pandas_market_calendars) to catch missing trading days, rather than just eyeballing gaps
The concept and mechanics of corporate actions — splits and dividends — and why they require adjusting historical prices to keep a return series continuous and comparable over time
The distinction between adjusted and unadjusted close prices, and why adjusted close is the "true" series for most analysis
Statistical/heuristic thinking for anomaly detection — defining what counts as a "suspicious" price jump versus normal volatility
Building a lightweight audit trail (a cleaning log) so every automated correction is traceable back to a reason
Working with a persistent SQLite database as the shared data layer across projects, including SQL query design (? placeholders over f-strings for safety)
### Notes
This project assumes the raw data pipeline already works (that was Project 1's job) and instead focuses entirely on trust: is the data actually correct, complete, and comparable across time? Nothing here touches strategy or signals yet — it's the "make sure the foundation isn't rotten" step that everything from Project 4 onward will quietly depend on. So far, the missing-trading-days script (missing_trading_days.py) has been built and walked through end to end; the split/dividend adjustment, suspicious-jump detection, and logging stages are still ahead.

---


## A closing note

This repository is meant to be worked top to bottom — each project assumes the code and understanding from everything before it. If you're using it to learn rather than just to build, follow the order rather than jumping to whatever looks interesting; the sequencing is the point. Suggestions and corrections are welcome as this evolves project by project.
