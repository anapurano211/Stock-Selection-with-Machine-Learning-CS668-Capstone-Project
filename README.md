# Stock Selection with Machine Learning — CS668 Capstone Project  
**Author:** Andrew Napurano  
**Course:** CS668 — Analytics Capstone Project, Pace University  
**Contact:** AN47692N@PACE.EDU  
**Repository:** [anapurano211/Stock-Selection-with-Machine-Learning-CS668-Capstone-Project](https://github.com/anapurano211/Stock-Selection-with-Machine-Learning-CS668-Capstone-Project)

---
# Project Proposal 

[Link to Proposal Presentation](https://docs.google.com/presentation/d/1aSppbj_hVKVHL4m_4S7yzVH2PhDZKpjp/edit?rtpof=true)

---


# Mid Semester Presentation Video 

[Link to Video](https://www.youtube.com/watch?v=z10mCpgJmm8)


---


# Mid Semester Presentation PPT

[Link to Mid Semester Presentation](https://docs.google.com/presentation/d/1aSppbj_hVKVHL4m_4S7yzVH2PhDZKpjp/edit?rtpof=true)


---


# Model Results

[Link to Model Results](https://docs.google.com/presentation/d/1aSppbj_hVKVHL4m_4S7yzVH2PhDZKpjp/edit?rtpof=true)


---

# Final Presentation

[Link to Final Presentation](https://www.youtube.com/watch?v=59tR4hmUAV0)


---

## 🎯 Project Overview
This project applies **machine learning and data analytics** to identify the company-level indicators that consistently drive stock outperformance relative to the **S&P 500** benchmark.  
According to S&P’s Global *SPIVA U.S. Scorecard*, over **97%** of actively managed domestic funds have underperformed the market’s average return (~8% annually over the last 20 years).  
The goal here is to demonstrate that a **systematic, data-driven model** built on fundamentals, technical indicators, and macroeconomic context can detect patterns of alpha generation beyond human intuition.

---

## 🧩 Research Questions
- Which **fundamental** and **technical** factors most strongly influence company outperformance over time?  
- Are there certain **time periods or sectors** where growth- or momentum-driven strategies dominate (e.g., Technology vs Defensive sectors)?  
- Can a trained **ML classification model** predict the probability of company outperformance and beat the S&P 500 on a risk-adjusted basis?

---

## 🗃️ Dataset Summary
- **Source:** PostgreSQL database (local), populated via **Financial Modeling Prep (FMP) API**.  
- **Coverage:** Historical S&P 500 constituents from **Jan 2015 – Sept 2025**.  
- **Size:** ~262,000 weekly records, >92 independent variables.  
- **Target Variable:**  
  - `Buy` (binary 1/0) → whether a stock outperformed the S&P 500 in the following week.  
- **Feature Groups:**  
  - **Fundamentals:** Sales Growth, Free Cash Flow Growth, Operating Margin, Leverage, Liquidity, Profitability.  
  - **Technical Indicators:** RSI, Moving Averages, Bollinger Bands, Momentum, Volatility, Volume.  
  - **Macro Factors:** CPI (Inflation), VIX, Oil, Gold, Treasury Yields.  
- **Data Hygiene:**  
  - All fundamentals lagged by **one quarter** to prevent look-ahead bias.  
  - Data ingested and versioned through an **Airflow ETL pipeline**.

---

## ⚙️ Methodology & Pipeline
1. **ETL & Database Automation:** Apache Airflow DAGs fetch, clean, and load data from FMP into PostgreSQL.  
2. **Feature Engineering:** Growth metrics, lag creation, normalization, winsorization.  
3. **EDA & Statistical Testing:** Decile analysis, hypothesis testing, and correlation analysis.  
4. **Modeling:** Logistic Regression, Random Forest, XGBoost, SVM, Deep Learning.  
5. **Deployment:** Streamlit app for user interaction and stock screening.

---

## 🤖 Machine Learning Roadmap
- Baseline logistic regression → ensemble → deep learning progression.  
- Evaluate ROC-AUC, F1, precision-recall.  
- Fama–MacBeth regressions for factor validation.  
- Decile portfolio backtests with transaction cost and turnover control.

---

## 🚀 Deliverables
- `df_eda`: unified dataset combining returns, fundamentals, and beats.  
- Reproducible notebooks + SQL scripts.  
- Executive PowerPoint (summary of findings).  
- Planned Streamlit deployment for interactive analysis.

---

## ⚡ Tech Stack
**Languages:** Python, SQL  
**Libraries:** pandas, NumPy, matplotlib, scikit-learn, XGBoost, yfinance, SQLAlchemy  
**Infra:** PostgreSQL, Apache Airflow, Streamlit  
**Data Source:** Financial Modeling Prep (FMP) API  

---


---

## 📬 Acknowledgments
This project bridges **data engineering, machine learning, and investment analytics** — transforming raw financial data into actionable quantitative insights.
