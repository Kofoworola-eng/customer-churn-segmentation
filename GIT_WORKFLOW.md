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