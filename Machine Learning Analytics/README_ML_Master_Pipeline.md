# 🧠 ML Master Pipeline — Stock Selection & Backtesting Framework

### Quantitative ML Pipeline for Predicting Stock Outperformance

This project trains multiple machine learning models to predict whether a stock will outperform its peers (e.g., deliver a higher next-period return).  
It then backtests those predictions as a **quantitative portfolio strategy** to evaluate real-world performance.

---

## 🚀 Features
- Rolling out-of-sample **time-series training**
- Supports **Logistic Regression, Random Forest, XGBoost, SGD, and Deep Learning**
- **Automatic model tuning** via time-series cross-validation
- **Portfolio backtesting** with return, volatility, Sharpe, and drawdown metrics
- **Feature importance** (model-based and permutation)
- **Diagnostics:** Confusion matrix, ROC/PR curves, generalization gap

---

## 🧩 Methodology

### 🕒 1. Rolling Training (Out-of-Sample)
For each **calendar test year**:
1. Train on the **previous 156 weeks** of data  
2. Standardize features (z-score within the train window)  
3. Tune hyperparameters with **time-series CV**  
4. Test strictly on the next year (no future leakage)

This mimics live trading: the model never sees the future.

---

### ⚙️ 2. Model Families

| Model | Type | Key Parameters | Notes |
|-------|------|----------------|-------|
| **Logistic Regression** | Linear | `C` (regularization strength) | Fast, interpretable baseline |
| **Random Forest** | Tree ensemble | `n_estimators`, `max_depth` | Captures non-linear patterns |
| **XGBoost** | Gradient boosting | `n_estimators`, `max_depth`, `eta` | Regularized, usually top performer |
| **SGD Classifier** | Linear SGD | `alpha`, `loss="log_loss"` | Lightweight stochastic baseline |
| **Deep Learning (Keras MLP)** | Neural net | 128–64–1 layers, dropout=0.2 | Learns complex interactions |

Each model outputs **`pred_proba`** = probability that a stock will beat the median peer performance next period.

---

### 💰 3. Prediction & Portfolio Backtest
For every test week:
1. Model outputs probabilities for all stocks  
2. Select **top q% (e.g., 5%)** highest scores  
3. Compute next-week returns using `sp500_weekly_rollups.ret_week_fwd1`  
4. Portfolio = average return of selected stocks  

Performance metrics:
| Metric | Definition |
|---------|-------------|
| **AnnReturn** | Annualized compound return |
| **AnnVol** | Annualized volatility |
| **Sharpe** | Risk-adjusted return |
| **MaxDD** | Maximum drawdown |

Cumulative returns are plotted vs. benchmark (e.g., S&P 500).

---

### 📊 4. Classification Metrics
For each model:
- Confusion Matrix (normalized by true labels)  
- Precision, Recall, F1-Score per class  
- ROC AUC and PR AUC  
- **Generalization Gap**: Train AUC − Test AUC (overfitting signal)

> 🧠  A large Train AUC ≫ Test AUC ⇒ overfit.  
> Small gap ⇒ model generalizes well.

---

### 🔍 5. Feature Importance

#### Model-based
- Linear models → coefficient magnitude  
- Trees → Gini gain or split importance  

#### Permutation-based
- Shuffle one feature at a time on test set  
- Drop in ROC AUC = importance

Permutation importance is **model-agnostic** and reflects *true out-of-sample* influence.

---

### 🤖 6. Deep Learning Notes
- Early stopping halts training when validation loss plateaus  
- Dropout(0.2) combats overfitting  
- Even if accuracy ≈ 50%, ranking power (AUC > 0.6) can still yield strong returns  
- Loss/validation curves help visualize convergence

---

### 📈 7. Economic Diagnostics
Evaluate *ranking power* (not just accuracy):

| Metric | Meaning | Ideal |
|---------|----------|-------|
| **Precision@Q / Lift@Q** | Fraction of true winners in top Q% | > 0.5 / Lift > 1 |
| **Decile Curve** | Avg return per score decile | Increasing trend |
| **Top–Bottom Spread** | Return(top) − Return(bottom) | Positive |
| **Information Coefficient (IC)** | Spearman( score, next return ) | Mean > 0, p < 0.05 |

These measure **economic signal quality**, not statistical fit.

---

## 🧮 Key Metrics Summary

| Metric | Definition | Ideal Behavior |
|---------|-------------|----------------|
| **Accuracy** | (TP + TN)/Total | ~50–60% typical |
| **ROC AUC** | Rank quality | > 0.6 good |
| **Sharpe** | Risk-adjusted return | > 1 strong |
| **Max DD** | Peak-to-trough loss | Lower = better |
| **AUC Gap** | Train–Test AUC | Small = generalizes well |

---

## 🧰 Core Functions

| Function | Description |
|-----------|-------------|
| `rolling_train_predict()` | Rolling yearly training & prediction |
| `_fit_logreg / _fit_rf / _fit_xgb / _fit_sgd / _fit_dl` | Model trainers |
| `_tune_inside_train()` | Lightweight internal CV tuner |
| `run_portfolio_backtest()` | Builds weekly portfolio & computes metrics |
| `run_model_end_to_end()` | Full pipeline: train → predict → backtest → diagnose |
| `run_all_models_and_summary()` | Runs all models + comparison tables |
| `feature_importance()` | Feature importance on latest test year |
| `fit_gap_by_year()` | Plots train/test AUC gap |
| `build_classification_summary()` | Aggregates classification metrics |

---

## 🧭 Typical Workflow

```python
import pandas as pd

# Load dataset
df = pd.read_parquet("cleaned_weekly.parquet")
ENGINE_URL = "postgresql://postgres:***@localhost:5432/SP500_ML"

# Run all models end-to-end
results, perf_table, summary = run_all_models_and_summary(
    df,
    engine_url=ENGINE_URL,
    model_list=["logreg", "rf", "xgb", "sgd", "dl"],
    q=0.05, min_names=30, tc_bps=0.0,
    rollup_table="sp500_weekly_rollups", ret_col="ret_week_fwd1"
)

print(perf_table)   # Portfolio performance comparison
print(summary)      # Classification summary per model
```

---

## 📉 Interpretation Guide

| Observation | Meaning | Suggested Action |
|--------------|----------|------------------|
| Train AUC ≫ Test AUC | Overfitting | Add regularization, reduce features |
| Train ≈ Test ≈ 0.5 | No signal | Add features, longer lookback |
| Test AUC > 0.6 | Predictive | Proceed to portfolio backtest |
| High Sharpe + Low DD | Robust | Validate in more years |
| RF/XGB > Linear | Non-linear relationships matter | Add interaction terms |
| DL best but unstable | High variance | Tune dropout, early stopping |

---

## 📚 Example Outputs
- **Model Comparison Table:** AnnReturn / AnnVol / Sharpe / MaxDD  
- **Confusion Matrices:** per-year & overall  
- **ROC & PR Curves:** visualize rank performance  
- **Feature Importance Plot:** top-N predictive drivers  
- **AUC Gap Plot:** train vs. test generalization  

---

## 💡 Summary
This project unifies machine learning and financial research best practices:

✅ Time-aware rolling backtests  
✅ Robust model comparison & diagnostics  
✅ Economic performance validation  
✅ Extendable framework for new features or models

It’s a **research-grade ML backtesting system** — ideal for exploring predictive signals, evaluating overfitting, and building data-driven portfolio strategies.

---

### 🧾 Citation
If used in academic or professional work, please cite as:
> *"Napurano, A. (2025). ML Master Pipeline — Stock Selection & Backtesting Framework."*

---

### 🛠️ Requirements
```
python>=3.9  
pandas  
numpy  
scikit-learn  
xgboost  
tensorflow (or keras)  
sqlalchemy  
matplotlib
```

---

### 🧩 License
MIT License © 2025 — for research and educational use.
