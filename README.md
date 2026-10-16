# 💸 From Churn Scores to Cash: Customer Churn & CLV Analysis with a Decision Memo

> A full analytics case study that ends in a recommendation and a dollar figure, not just an F1 score.

![Python](https://img.shields.io/badge/Python-3.11-blue) ![SQL](https://img.shields.io/badge/SQL-DuckDB-yellow) ![XGBoost](https://img.shields.io/badge/XGBoost-model-green) ![SHAP](https://img.shields.io/badge/SHAP-explainability-purple) ![PowerBI](https://img.shields.io/badge/Power%20BI-dashboard-F2C811)

**Dashboard:** `<Power BI / Tableau Public link>` · **Decision memo:** [`memo/retention_recommendation.pdf`](memo/)

---

## 1. Overview

A retailer wants to know: **which customers are likely to leave, why, and is it worth spending money to keep them?**

This project walks the whole analyst workflow:

1. **Understand** customer behaviour (RFM segments, cohorts, funnel)
2. **Predict** churn with gradient boosting and explain it with SHAP
3. **Value** customers with a lifetime-value model
4. **Decide**: simulate a retention campaign and estimate net gain by targeting threshold
5. **Communicate**: interactive dashboard plus a one-page memo a manager could act on

## 2. Business question

> "If we run a retention campaign costing $C per contacted customer, whom should we target, and what net gain should we expect?"

## 3. Dataset

**Online Retail II** (UCI): ~1M invoice lines from a UK online retailer, 2009-2011, with customer IDs and countries. It suits RFM, cohorts and CLV because customers buy repeatedly. 

## 4. Methodology

### 4.1 Data prep (SQL + Python)
- Remove cancellations (invoices starting with "C") into a returns table; drop null customer IDs; handle outliers
- Build `customer_snapshot` table at observation date(s): features + 90-day-forward label

### 4.2 Exploratory analysis
- **RFM segmentation:** Recency, Frequency, Monetary → quintile scores → named segments (Champions, Loyal, At Risk, Hibernating...)
- **Cohort retention:** monthly acquisition cohorts, retention heatmap
- **Funnel:** (customers → repeat buyers → 3+ orders → high-value) drop-off rates

### 4.3 Churn model
- Features: recency, frequency, monetary, average order value, tenure, days between orders (mean/std), return rate, product diversity, country, trend in last 90 vs prior 90 days
- Models: logistic regression (baseline), XGBoost/LightGBM (main)
- **Time-based split** (train on earlier snapshots, test on a later one), not random
- Metrics: ROC-AUC, PR-AUC, precision@top-decile, calibration curve
- **SHAP:** global importance + a few individual customer explanations

### 4.4 CLV
- Probabilistic: **BG/NBD + Gamma-Gamma** (`lifetimes` or `PyMC-Marketing`) for expected purchases and spend over 12 months
- Validate on a holdout period (predicted vs actual spend by decile)

### 4.5 Campaign value simulation

```
expected_gain(customer) = P(churn) × P(saved | contacted) × CLV_12m × margin
net_gain(threshold)     = Σ expected_gain − (contact_cost × customers_contacted)
```

- Sweep targeting from top 1% to 100% risk; plot net gain vs % targeted
- Choose the optimum and show sensitivity to the assumptions (save rate 5-25%, cost, margin)
- **Be transparent:** save rate and margin are assumptions; state them and show a range, ideally with a simple A/B design recommendation to measure the real uplift

## 5. Deliverables

| Deliverable | Description |
|---|---|
| `notebooks/` | 01_cleaning, 02_eda_rfm_cohorts, 03_churn_model, 04_clv, 05_campaign_sim |
| `sql/` | Feature and snapshot queries |
| `dashboard/` | `.pbix` (or Tableau workbook), screenshots, published link |
| `memo/retention_recommendation.pdf` | One page: problem, finding, recommendation, expected impact, risks, next step |
| `src/` | Reusable feature + modelling code |

### Dashboard pages
1. **Executive summary:** KPIs, churn rate, revenue at risk, campaign ROI
2. **Segments:** RFM treemap, segment value
3. **Retention:** cohort heatmap, funnel
4. **Risk & drivers:** risk deciles, SHAP top drivers
5. **Campaign simulator:** parameter inputs (cost, save rate, margin) → net gain curve


## 6. Repository structure

```
churn-clv-decision/
├── data/                 # raw not committed
├── sql/
├── notebooks/
├── src/
│   ├── features.py
│   ├── train.py
│   ├── clv.py
│   └── campaign_sim.py
├── dashboard/
├── memo/
├── reports/figures/
├── requirements.txt
└── README.md
```

## 7. Tech stack

| Area | Tools |
|---|---|
| Languages | Python, SQL |
| Data | DuckDB or PostgreSQL, pandas |
| ML | scikit-learn, XGBoost/LightGBM, SHAP, imbalanced-learn (optional) |
| CLV | lifetimes / PyMC-Marketing |
| BI | Power BI (or Tableau Public) |
| Docs | Markdown, PDF memo |

## 8. Results

| Metric | Value |
|---|---|
| Churn rate (90-day) | _x%_ |
| Baseline model ROC-AUC / PR-AUC | _x / x_ |
| XGBoost ROC-AUC / PR-AUC | _x / x_ |
| Precision @ top 10% | _x%_ |
| Top 3 churn drivers (SHAP) | _..._ |
| Optimal targeting threshold | _top x%_ |
| Estimated net gain (base case, range) | _£X (A-B)_ |
