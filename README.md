# Which product categories generate the most returns, and is it worth acting on?
IT135IU – Introduction to Data Science, course project.

## Team
| Name | Student ID | Role |
|---|---|---|
| DANG UYEN THU | ITDSIU25044 | Product owner |
| TRUONG THUAN TUAN | ITDSIU25051 | Data engineer |
| NGUYEN AN KHOI | ITDSIU25016 | Data analyst |
| NGUYEN THI KHANH LINH | ITDSIU25019 | Visualisation lead |
| NGUYEN CHI PHUC | ITDSIU25028 | Reproducibility lead |
| DANG LE TRUC LINH | ITDSIU25018 | Ethics and privacy officer |

## Decision question
Which product categories generate the most returns, and is it worth acting on?

## Data
- **Source 1:** Online Retail — https://archive.ics.uci.edu/dataset/352/online+retail — UCI Machine Learning Repository, donated by Daqing Chen — 541,909 rows, covers 01/12/2010–09/12/2011
- **Source 2:** Online Retail II — https://archive.ics.uci.edu/dataset/502/online+retail+ii — UCI Machine Learning Repository, donated by Daqing Chen — 1,067,371 rows, covers 01/12/2009–09/12/2011
- **Publisher:** UCI Machine Learning Repository
- **Retrieved:** 9/14/2026
- **License:** Creative Commons Attribution 4.0 International (CC BY 4.0), stated explicitly on both dataset pages

> Note: both datasets are from the same retailer and their date ranges overlap
> almost completely (Online Retail II's second sheet already covers the whole
> period of the original Online Retail dataset, plus one extra earlier year).
> The working dataset is built by concatenating both and explicitly dropping
> duplicate transactions from the overlapping period — this decision is
> logged in `data/processed/cleaning_log.csv` and should be stated plainly in
> Section 2 of the report.

> Cleaning decision: negative quantities and prices are retained because they
> represent transaction reversals or corrections. They contribute to net
> revenue. The derived `line_total` is transaction revenue, not profit, because
> the source data does not include product cost.

## How to run
1. Download `Online Retail.xlsx` from the Source 1 URL and `online_retail_II.xlsx` from the Source 2 URL into `data/raw/` (this folder is git-ignored; do not commit the raw files or upload them to Blackboard — link to the URLs above instead).
2. `pip install -r requirements.txt`
3. Open `notebooks/D2_OnlineRetail_DataEngineering_Starter.ipynb` and run all cells top to bottom.
4. Output appears in `data/processed/`: `working_dataset.csv`, `data_profile.csv`, `cleaning_log.csv`.

## Random seed
`RANDOM_SEED = 42`, set at the top of every notebook, used wherever sampling or splitting occurs.

## Repository structure
```
data/
  raw/          # untouched .xlsx files (not committed — see .gitignore)
  processed/    # working_dataset.csv, data_profile.csv, cleaning_log.csv
notebooks/      # D2, D3 analysis notebooks
report/         # final PDF report and slides
figures/        # exported charts used in the report / slides
README.md
requirements.txt
```

## AI tool use
We used Claude to learn how to use new Python libraries and understand Git commands and workflows. The AI output was used mainly for explanations, examples, and guidance while learning. We checked the suggested code and commands by running them myself and verifying that they worked correctly before including them in the project.
