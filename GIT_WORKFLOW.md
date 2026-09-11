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