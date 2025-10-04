
# SP500_ML Database — Overview & Table Guide

This README gives a **plain‑English description** of the datasets in your Postgres warehouse.  
Focus: what each table/view contains, the row **grain**, important/typical fields, and **how it’s used downstream**. No primary‑key details are included.

---

## Contents
- [Universe & Membership](#universe--membership)
- [Prices & Technicals](#prices--technicals)
- [Weekly Aggregates & Targets](#weekly-aggregates--targets)
- [Fundamentals (Income Statement, Cash Flow, Balance Sheet)](#fundamentals-income-statement-cash-flow-balance-sheet)
- [Market Reference Data](#market-reference-data)
- [Risk/Volatility](#riskvolatility)
- [Sectors & Metadata](#sectors--metadata)
- [Common Join Patterns](#common-join-patterns)

---

## Universe & Membership

### `sp500_long_latest_profiles`
**What it is:** A long history of S&P 500 membership snapshots (point‑in‑time).  
**Grain:** One row per **(date, ticker)** snapshot.  
**Important fields:**
- `date` – snapshot date
- `ticker` – the member ticker on the snapshot date
- `latest_ticker` – your **canonical** (current) ticker for that company, used to stitch histories across ticker changes

**Use it for:**
- Building point‑in‑time S&P universes (e.g., “members as of 2020‑06‑30”)
- Mapping legacy tickers to the current “latest_ticker” used throughout the DB

---

## Prices & Technicals

### `sp500_prices_daily_yahoo`
**What it is:** Daily OHLCV from Yahoo Finance.  
**Grain:** One row per **(date, ticker_latest)**.  
**Important fields:** `date, ticker_latest, open, high, low, close, adj_close, volume`

**Use it for:** raw pricing, backtests, and as the base for weekly rollups and momentum features.

### `sp500_prices_technicals_daily`
**What it is:** Daily **technicals** computed from prices.  
**Grain:** One row per **(date, ticker_latest)** (last trading day in each week is often used).  
**Typical fields:**
- Momentum windows: `ret_30d` (≈1m), `ret_180d` (≈6m), `ret_360d` (≈12m)
- RSI: `rsi_14`, `rsi_9`, `rsi_3`
- Moving averages: `sma_50`, `sma_100`, `sma_200`
- Bollinger: `bb_lower, bb_middle, bb_upper, bb_bandwidth, bb_percent`
- Beta & market ref: `beta_12m`, and optional `mkt_1m / mkt_6m / mkt_12m` (SPX trailing returns aligned by date)

**Use it for:** cross‑sectional momentum studies, feature engineering for weekly models.

### `v_sp500_daily_features` (view)
**What it is:** A convenience **view** that joins daily price/technical features (and sometimes β/market returns) for the latest date within each week.  
**Use it for:** pulling a single “as‑of week” feature row per ticker for model training labels.

---

## Weekly Aggregates & Targets

### `sp500_weekly_rollups`
**What it is:** Weekly price aggregates and **forward targets** computed from daily prices **within S&P membership weeks**.  
**Grain:** One row per **(week_end, ticker_latest)** where the ticker is a member that week.  
**Fields (typical):**
- Weekly aggregates: `adj_close_week_last`, `adj_close_week_avg`, `volume_week_sum`  
- Dollar volume proxy: `dollar_vol_week_avgp` (avg weekly price × weekly volume)  
- Weekly return & next‑week return: `ret_week`, `ret_week_fwd1`
- Cross‑sectional median next week: `median_ret_fwd1`
- Binary label: `target_gt_median` (next‑week return > next‑week cross‑sectional median)

**Use it for:** classification/regression targets and leakage‑free training sets.

### `sp500_weekly_vs_sp500` *(if present)*
**What it is:** Same weekly grain, but the label compares **next‑week return vs the S&P 500** next‑week return.  
**Label:** e.g., `target_gt_sp500`.

---

## Fundamentals (Income Statement, Cash Flow, Balance Sheet)

These are pulled from FMP and stored **quarterly** with **validity windows** so you can do “as‑of” joins.

### `income_statements_q`
**What it is:** Quarterly **income statements** with **TTM** sums and growths.  
**Grain:** One row per **(symbol, date)** **filing period**; also includes `date_start/date_end` for validity.  
**Core fields:** revenue, costOfRevenue, operatingIncome, netIncome, EPS, margins, etc.  
**Derived fields (typical):**
- TTM levels: `revenue_ttm`, `operatingincome_ttm`, `netincome_ttm`
- TTM growth (signed, YoY of TTM): `revenue_ttm_growth`, `operatingincome_ttm_growth`, `netincome_ttm_growth`
- **Quarter‑over‑quarter YoY** (same quarter last year): `revenue_q_yoy`, `operatingIncome_q_yoy`, `netIncome_q_yoy`
- (In analysis frames you often compute 1/2/3‑year lags of TTM via `LAG(..., 4/8/12)`)

**Use it for:** sales/earnings growth factors, TTM margins, and profit‑based yields.

### `cashflow_statements_q`
**What it is:** Quarterly **cash‑flow statements** with **TTM** and growth.  
**Grain:** One row per **(symbol, date)**; has validity windows.  
**Core fields:** operatingCashFlow, capitalExpenditure, freeCashFlow, investing/financing lines.  
**Derived fields:**  
- TTM levels: `operatingcashflow_ttm`, `freecashflow_ttm`, `investingcashflow_ttm`  
- TTM YoY growth (signed): `operatingcashflow_ttm_growth`, `freecashflow_ttm_growth`, `investingcashflow_ttm_growth`  
- *(Optional in analysis)* Quarter‑over‑quarter YoY growth for these metrics.

**Use it for:** cash yield factors (FCF/EV, OCF/MarketCap), quality screens.

### `balance_sheets_q`
**What it is:** Quarterly **balance sheets** with **YoY** changes and many **ratios**.  
**Grain:** One row per **(symbol, date)**; has validity windows.  
**Core fields:** assets/liabilities/equity and sub‑components (current assets, debt, netDebt, etc.).  
**Derived fields (examples):**
- YoY (signed) on key lines: `totalAssets_yoy`, `totalLiabilities_yoy`, `totalStockholdersEquity_yoy`, `totalDebt_yoy`, `netDebt_yoy`, etc.
- Liquidity: `current_ratio`, `quick_ratio`, `cash_ratio`, `working_capital`, `working_capital_to_assets`, `inventory_to_current`
- Leverage & capital structure: `debt_to_equity`, `debt_to_assets`, `net_debt_to_equity`, `liabilities_to_assets`, `equity_ratio`, `lt_debt_to_capital`, `total_debt_to_capital`
- Financial leverage variants: `financialLeverage`, `financialLeverage_avg4q`, `financialLeverage_yoy`

**Use it for:** balance‑sheet quality, leverage constraints, and cross‑sectional risk controls.

---

## Market Reference Data

### `market_caps_d`
**What it is:** Daily **market cap** snapshots.  
**Grain:** (symbol, date).  
**Field:** `marketcap`.

### `enterprise_values_q`
**What it is:** Quarterly **enterprise value** snapshots (from FMP EV series).  
**Grain:** (symbol, date).  
**Field:** `enterprisevalue`.

**Use both for:** valuation factors (e.g., `sales_ttm_to_ev`, `fcf_ttm_to_ev`, `ni_ttm_to_mcap`, book‑to‑market, etc.).

---

## Risk/Volatility

### `realized_vol_d`
**What it is:** Daily realized volatility summaries.  
**Grain:** (symbol, date).  
**Fields:** `vol_1m_ann`, `vol_6m_ann`, `vol_12m_ann` (annualized).

**Use it for:** risk screens, portfolio sizing, and as a control in modeling.

---

## Sectors & Metadata

### `v_sp500_sector_clean` (view)
**What it is:** Clean mapping from ticker to **sector**.  
**Grain:** one row per ticker.  
**Field:** `sector_clean`.

**Use it for:** sector‑aware EDA (group means, decile hit‑rates) & modeling (one‑hot/embedding).

---

## Common Join Patterns

Below are typical point‑in‑time joins used in your feature pipelines. Replace table/view names as needed.

- **Weekly target + “as‑of” daily features**
  ```sql
  SELECT t.week_end, t.ticker_latest, t.target_gt_median, f.*
  FROM sp500_weekly_rollups t
  LEFT JOIN LATERAL (
    SELECT *
    FROM v_sp500_daily_features d
    WHERE d.ticker_latest = t.ticker_latest
      AND d.date <= t.week_end
      AND d.date >  t.week_end - INTERVAL '6 days'
    ORDER BY d.date DESC
    LIMIT 1
  ) f ON TRUE;
  ```

- **Attach income/cashflow/balance‑sheet (validity windows)**
  ```sql
  LEFT JOIN LATERAL (
    SELECT revenue_ttm, netincome_ttm, operatingincome_ttm, revenue_ttm_growth
    FROM income_statements_q i
    WHERE i.symbol = t.ticker_latest
      AND t.week_end >= i.date_start
      AND t.week_end <  i.date_end
    LIMIT 1
  ) i ON TRUE
  ```

- **Attach valuation levels “as of” feature date**
  ```sql
  LEFT JOIN LATERAL (
    SELECT marketcap AS size_mcap
    FROM market_caps_d mc
    WHERE mc.symbol = t.ticker_latest
      AND mc.date <= f.date
    ORDER BY mc.date DESC
    LIMIT 1
  ) mcap ON TRUE
  LEFT JOIN LATERAL (
    SELECT enterprisevalue
    FROM enterprise_values_q ev
    WHERE ev.symbol = t.ticker_latest
      AND ev.date <= f.date
    ORDER BY ev.date DESC
    LIMIT 1
  ) ev ON TRUE
  ```

---

## Notes & Conventions
- **Ticker field:** modeling tables generally use `ticker_latest` (your canonical key) while vendor fundamentals use `symbol`. Treat them as equivalent for joins.
- **Validity windows:** fundamentals tables include `date_start`/`date_end` so that **as‑of** joins never peek forward.
- **TTM & YoY:** TTM are strict sums over the last 4 quarters. YoY fields compare **current vs 4 quarters prior** (same quarter last year). For signed growth, the symmetric formula `2*(a-b)/(abs(a)+abs(b))` is used to handle zeros/negatives.
- **Weekly grain:** `week_end` is the Friday of the week; when merging daily features, pick the **last trading day within that week**.

---

### Questions / Next steps
If you’d like this exported as a Confluence page, a PDF, or expanded into a **column‑level data dictionary** (with min/max/NULL %, sample values), I can generate that from the live database.
