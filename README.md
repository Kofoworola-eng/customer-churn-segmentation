# Customer Segmentation & Churn Prediction

An end-to-end data analytics project that identifies at-risk customer segments and predicts 
churn using RFM analysis and machine learning, built on the AdventureWorks2022 database.

## Business Question

Which customers are at risk of churning, and what behavioral patterns predict it, so the 
business can prioritize retention efforts effectively?

## Why This Project

Churn analysis is one of the most valuable and most misunderstood skills in data work. It is 
not just "run a model and get a number", it requires understanding customer behavior patterns, 
avoiding common pitfalls like data leakage, and translating a prediction into a decision someone 
can actually act on.

This project is documented step by step (see `GIT_WORKFLOW.md`) so that early-career analysts 
and aspiring data professionals can follow the real build process, including the mistakes and 
fixes along the way, not just a polished final result.

## Approach

**Pipeline:** SQL (extraction) → CSV → Python (cleaning, analysis, modeling)

1. **Data extraction** : SQL queries against AdventureWorks2022, scoped to individual (retail) 
   customers only
2. **RFM feature engineering** : Recency, Frequency, Monetary calculated per customer, plus 
   Average Order Value and Average Purchase Gap
3. **Customer segmentation** : quintile-based RFM scoring, translated into business-readable 
   segments (Champions, Loyal Customers, At Risk, Lost, etc.)
4. **Churn prediction modeling** : Logistic Regression baseline, then Random Forest, with 
   hyperparameter tuning via GridSearchCV
5. **Feature importance analysis** : identifying which behaviors actually predict churn

## Key Findings

- **Customer base split:** 19,119 unique customers, with a 54% active / 46% churned split 
  (using a 180-day inactivity threshold)
- **Segment distribution:** Loyal Customers is the largest group (4,655), followed by Needs 
  Attention, Lost, New/Promising, Champions, and At Risk
- **Retention priority group:** At Risk + Lost customers total 5,630 (~29% of the customer base)
- **Data leakage caught and fixed:** an early feature (Tenure) inflated model accuracy to 91% 
  by indirectly leaking the churn label for one-time buyers. This was investigated, confirmed, 
  and removed — a deliberate example of catching a common but serious mistake rather than 
  reporting an inflated result
- **What predicts churn:** Average Order Value, Monetary value, and Average Purchase Gap 
  together account for ~90% of predictive power. Purchase frequency and customer location have 
  minimal impact
- **Final model performance (tuned Random Forest):** 64.6% accuracy, 59.3% precision, 72.6% 
  recall, 65.3% F1 — tuned deliberately to favor recall, since missing an at-risk customer is 
  typically costlier to a business than a false alarm

## Business Recommendation

Retention efforts should prioritize customers with **declining or inconsistent purchase 
patterns and lower average order values**, rather than focusing purely on inactivity alone. 
The At Risk and Lost segments (29% of the customer base) represent the clearest opportunity 
for targeted retention campaigns.

## Tech Stack

- **SQL** : extraction from AdventureWorks2022
- **Python** : pandas, scikit-learn, matplotlib, seaborn
- **Modeling** : Logistic Regression, Random Forest, GridSearchCV

## Project Structure
customer-churn-segmentation/
├── sql/ → extraction queries
├── data/
│ ├── raw/ → raw SQL exports (not tracked in git)
│ └── processed/ → cleaned/feature-engineered data (not tracked in git)
├── notebooks/
│ ├── 01_data_loading_exploration.ipynb
│ ├── 02_rfm_segmentation.ipynb
│ ├── 03_visualization.ipynb
│ └── 04_churn_prediction_model.ipynb
├── models/ → saved trained model + scaler
├── dashboard/ → visualization assets
├── GIT_WORKFLOW.md → full step-by-step build log
└── README.md


## How to Run

1. Clone the repo
2. Install dependencies: `pip install pandas scikit-learn matplotlib seaborn joblib`
3. Run the notebooks in order (01 → 04)

## Full Build Log

See [`GIT_WORKFLOW.md`](./GIT_WORKFLOW.md) for the complete step-by-step process, including 
decisions, errors encountered, and how they were resolved.
