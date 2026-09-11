## Project Setup

1. Created GitHub repo `customer-churn-segmentation` (public, README + Python .gitignore template)
2. Cloned locally: `git clone https://github.com/kofoworola-eng/customer-churn-segmentation.git`
3. Created folder structure: sql/, data/raw/, data/processed/, notebooks/, src/, models/, dashboard/
4. Extended .gitignore to exclude data/raw/*.csv and data/processed/*.csv (raw data shouldn't 
   live in git history/repo size), added .gitkeep files so empty folders still show on GitHub
5. First commit: "Initial project structure"
6. Note: GitHub auto-corrected a username casing mismatch (kofoworola-eng vs Kofoworola-eng) 
   on push — updated remote URL to match: 
   `git remote set-url origin https://github.com/Kofoworola-eng/customer-churn-segmentation.git`

   ## Step 2-3: SQL Extraction & Export

Wrote extraction query against AdventureWorks2022, joining Sales.Customer to Person.Person 
(filters to individual/retail customers only, since PersonID is null for reseller accounts) 
and Sales.SalesOrderHeader (order history), with Sales.SalesTerritory for regional segmentation.

Query pulled 31,465 rows: CustomerID, FirstName, LastName, SalesOrderID, OrderDate, TotalDue, TerritoryName.

Exported via SSMS "Save Results As" to data/raw/customer_orders_raw.csv

Gotcha: first export had no header row, and manually typing headers into the CSV corrupted 
the OrderDate column formatting. Fix: enabled "Include column headers when copying or saving 
results" in SSMS (Tools > Options > Query Results > SQL Server > Results to Grid) and 
re-exported cleanly instead of editing the file by hand.

## Step 4: Data Loading & Initial Exploration

Set up notebooks/01_data_loading_exploration.ipynb (Python 3.12.3 kernel).
Installed ipykernel (`pip install ipykernel`) to enable notebook execution in VS Code.

Loaded data/raw/customer_orders_raw.csv with pandas, parsing OrderDate as datetime on load.

Validation:
- Shape: (31465, 7) - matches SQL extraction row count exactly
- No null values in any column
- OrderDate correctly typed as datetime64[ns]
- All other dtypes as expected (int64 for IDs, float64 for TotalDue, object for names/territory)

Clean dataset, no null-handling needed before moving to feature engineering.

## Step 5-6: RFM Feature Engineering & Churn Labeling

Aggregated 31,465 order-level rows down to 19,119 unique customers using groupby('CustomerID').

Calculated RFM features:
- Recency: days since last order, relative to snapshot date (2014-06-30)
- Frequency: count of orders per customer
- Monetary: sum of TotalDue per customer

Distribution notes: median Frequency is 1 order (long tail up to 28), Monetary ranges from 
$1.52 to $989,184, Recency ranges from 0 to 1,126 days.

Created churn label: Churned = 1 if Recency > 180 days (6-month window), else 0.

Result: 10,354 active (54%) vs 8,765 churned (46%) - a well-balanced split, meaning no class 
imbalance handling (e.g. SMOTE) needed for the prediction model.

## Step 7-8: RFM Scoring & Segmentation

Converted Recency, Frequency, Monetary into 1-5 quintile scores using pd.qcut (R_Score reversed 
since low Recency is good; F_Score uses rank(method='first') to handle heavy value ties from 
the low median order count).

Combined scores into a business-readable Segment label via a rule-based function:
Champions, Loyal Customers, New/Promising, At Risk, Lost, Needs Attention.

Segment distribution:
- Loyal Customers: 4,655
- Needs Attention: 3,542
- Lost: 3,263
- New/Promising: 2,838
- Champions: 2,454
- At Risk: 2,367

Insight: At Risk + Lost = 5,630 customers (~29% of base), representing the priority group 
for retention efforts.

## Notebook Reorganization

Split the growing single notebook into purpose-specific notebooks for readability:
- 01_data_loading_exploration.ipynb - loading + initial validation only
- 02_rfm_segmentation.ipynb - RFM calculation, scoring, segmentation logic; saves output 
  to data/processed/rfm_segmented.csv
- 03_visualization.ipynb - loads processed data, builds charts (kept separate from analysis 
  to model a clean separation of concerns)

## Step 9: Segment Distribution Visualization

Installed matplotlib and seaborn (pip install matplotlib seaborn).

Built a bar chart of customer counts by RFM segment, ordered largest to smallest, 
saved to dashboard/segment_distribution.png for use in README and future posts.

Chart confirms segment sizes: Loyal Customers (4,655) is the largest group, followed by 
Needs Attention, Lost, New/Promising, Champions, and At Risk.

## Step 10: Feature Engineering for Churn Prediction

Created notebooks/04_churn_prediction_model.ipynb. Installed scikit-learn (pip install scikit-learn).

One-hot encoded TerritoryName using pd.get_dummies (drop_first=True to avoid redundant columns).

Selected features: Frequency, Monetary, and territory dummy columns.

Important: deliberately excluded Recency, R_Score, F_Score, M_Score from the feature set. 
Since Churned was derived directly from Recency (>180 days), including Recency-based features 
would cause data leakage - the model would learn to look up the label rather than find real 
predictive patterns. This is a common mistake worth avoiding explicitly.

## Step 11: Train/Test Split and Baseline Model

Split data 80/20 (train_test_split, random_state=42, stratify=y to preserve the 54/46 
churn ratio in both sets). Train: 15,295 rows, Test: 3,824 rows.

First model attempt (Logistic Regression) hit a ConvergenceWarning - caused by Monetary being 
on a much larger scale than Frequency and the territory dummy columns, which destabilizes the 
optimization. Fixed by applying StandardScaler to normalize all features to the same scale 
(mean 0, std 1) before training.

Re-trained Logistic Regression on scaled features - converged cleanly, no warnings.

## Step 12: Baseline Model Evaluation

Evaluated Logistic Regression baseline on test set:
- Accuracy: 55.0%
- Precision: 50.7%
- Recall: 67.8%
- F1 Score: 58.0%

Confusion matrix: [[916, 1155], [564, 1189]]

Honest assessment: baseline barely beats the naive "predict majority class" benchmark (~54%, 
matching the churn split). Likely cause: Frequency and Monetary alone are historical totals, 
not strong predictors of *future* churn behavior. Deliberately excluding Recency avoided data 
leakage, but also removed most of the available signal.

Decision: improve the feature set before trying a more complex model, since better features 
typically beat a fancier algorithm on weak inputs. Planned additions (non-leaking):
- Tenure: days between first order and snapshot date
- Average Order Value: Monetary / Frequency
- Purchase gap variability: average days between orders for repeat customers

## Step 13: Engineered Additional Features

Added three new features to 02_rfm_segmentation.ipynb, inserted right after the base RFM 
calculation:
- Tenure: days between first order and snapshot date (customer relationship length)
- AvgOrderValue: Monetary / Frequency (spend per transaction, not total)
- AvgPurchaseGap: average days between orders for repeat customers (0 for one-time buyers)

Re-ran the full notebook top to bottom so downstream steps (churn label, RFM scoring, 
segmentation) recalculated correctly on the updated rfm table. Overwrote 
data/processed/rfm_segmented.csv with the expanded feature set.

## Step 14: Feature Set Expansion and a Critical Data Leakage Discovery

Added Tenure, AvgOrderValue, AvgPurchaseGap to the feature set and re-trained. Accuracy jumped 
from 55% to 91% - an unrealistically large improvement, treated as a red flag rather than a win.

Investigated and confirmed leakage: for one-time buyers (11,649 of 19,119 customers, ~61% of 
the base), FirstOrderDate equals LastOrderDate, meaning Tenure was numerically identical to 
Recency for this entire group. Since Recency directly determined the Churned label, Tenure was 
indirectly leaking the answer to the model.

Correlation check confirmed it: Tenure/Recency correlation 0.38, with 11,649/11,649 one-time 
buyers showing Tenure == Recency exactly.

Fix: removed Tenure from the feature set entirely.

Result after fix: Accuracy 54.4%, Precision 50.3%, Recall 59.2%, F1 43.7% - back in line with 
the original baseline.

Key finding: Frequency, Monetary, AvgOrderValue, and AvgPurchaseGap alone do not carry strong 
predictive signal for churn in this dataset once leakage is properly removed. This is an honest 
and valuable result - a clean 54% is more useful than a leaked 91%, since the leaked version 
would fail immediately in a real deployment.
