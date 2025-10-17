# The Key to Growth

1. **Database Connection:** Connect to a local PostgreSQL database using SQLAlchemy (`ENGINE_URL = "postgresql://postgres:CSDBMS623@localhost:5432/SP500_ML"`).  
2. **Anchor Date Setup:** Define `as_of` (evaluation date), 3-year lookback `anchor`, and a small `buffer_days` to handle weekends or holidays.  
3. **Universe Query:** Pull distinct active S&P 500 tickers from `sp500_long_latest_profiles` as of the `as_of` date.  
4. **Price Data Extraction:** Retrieve adjusted daily close prices from `sp500_prices_daily_yahoo` between `anchor - buffer_days` and `as_of`.  
5. **3-Year Return Calculation:** Compute per-ticker start/end prices around the anchor window to calculate `ret_3y = (px_end / px_start - 1)`.  
6. **Benchmark Return:** Download the S&P 500 (`^GSPC`) or SPY (total return) via `yfinance`; compute `mkt_ret_3y`.  
7. **Relative Alpha Calculation:** Compare stock vs market (`alpha_3y = ret_3y - mkt_ret_3y`), flag `beat_sp500_3y` = 1 if alpha > 0.  
8. **Income Statement Pull:** Query `income_statements_q` for TTM fundamentals and 12-month lags (`LAG()` window). Extract revenue, net income, and operating income.  
9. **Cash Flow Statement Pull:** Query `cashflow_statements_q` for TTM operating and free cash flow + lag12 versions.  
10. **Balance Sheet Pull:** Query `balance_sheets_q` for total assets and long-term debt with 12-month lags.  
11. **Earnings Surprises:** Aggregate analyst beat magnitudes ≥10% from `earnings_surprises_q` since 2022-09-29.  
12. **Growth Calculations:** Use helper functions (`pct_growth_plain_pospos`, `pct_growth_signed`, `growth_with_fallback`) to compute YoY % growth.  
13. **Winsorization:** Clip growth metrics to the 1st and 99th percentile to remove outliers.  
14. **Merging Fundamentals:** Join income, cash flow, and balance sheet growth tables into the unified DataFrame (`df_eda`).  
15. **Earnings Beat Integration:** Merge beat counts per symbol into `df_eda`.  
16. **Lag Handling:** Create lag columns using `add_group_lag()` for each symbol/date combination.  
17. **Growth Batch Application:** Apply growth functions across multiple variable pairs (`revenue_ttm`, `netincome_ttm`, `operatingincome_ttm`, etc.).  
18. **Data Cleaning:** Drop NaNs, replace ±inf with NaN, enforce numeric dtype, and filter only valid finite rows.  
19. **Decile Binning:** Use `pd.qcut()` to assign deciles by revenue growth and other key factors.  
20. **Visualization Setup:** Build heat-mapped bar charts with `TwoSlopeNorm` to highlight above/below mean return patterns.  
21. **Final Analytical Dataset:** Export the resulting DataFrame (`df_eda`) as the foundation for exploratory and ML modeling.  

## 🧩 Research Questions
- Which **fundamental** and **technical** factors most strongly influence company outperformance over time?  
- Are there certain **time periods or sectors** where growth- or momentum-driven strategies dominate (e.g., Technology vs Defensive sectors)?  

---

## 📊 Key Analytical Findings (2022–2025)
### 1. Sales Growth is the Primary Driver  
- Top-decile sales growth firms outperformed the bottom decile by ~600%.  
- 46% of variance in returns explained by sales growth.  

### 2. Profitability and Earnings Efficiency Matter  
- Positive operating margin growth correlated with strong alpha, especially in Tech and Industrials.  
- Earnings surprises boosted returns except in rate-sensitive sectors (Real Estate).  

### 3. Momentum Sustains Performance  
- High momentum in prior years led to future outperformance (93.5% → 35% sequential returns).  
- Outperformers spent more time above their 200-day moving average (56 vs 27 days).  

### 4. Free Cash Flow Growth Correlates with Alpha  
- Top-decile FCF growth firms outperformed by 336%.  
- T-Test p=0.06 suggests near-significant differentiation; macro volatility likely cause.  

### 5. Income Growth & Earnings Power  
- Income growth difference: 106.4% vs 25.8%; Cohen’s D=0.41.  
- Higher earnings power = greater probability of S&P 500 outperformance.  

---

## 📈 Visual Analytics
- Decile bar charts (3Y returns by revenue or cash flow growth).  
- Heat-mapped returns (`TwoSlopeNorm`) for above/below-mean comparisons.  
- Sector-level visuals for cross-sectional insights.


---

## 🧭 Interpretation & Next Steps
The findings confirm that **revenue, profitability, and liquidity growth** are core signals of stock outperformance. Future steps include:  
- Integrating valuation factors (PE, PB, PS).  
- Expanding the backtesting framework with rolling re-trains.  
- Adding explainability (SHAP) and sector rotation simulations.


