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

> The source data has no product `Category` column. The D2 notebook derives one
> from `Description` using ordered keyword rules (for example, mugs and cups map
> to Kitchen & Dining, while candles map to Home Decor). This is a heuristic
> grouping, not an official product taxonomy; unmatched descriptions are labeled
> `Other / Unclassified`, and the first matching rule wins. For return analysis,
> negative `Quantity` values indicate returned/reversed quantities; invoices
> marked as cancellations are tracked separately by `is_cancellation`.


## How to run
This repo does **not** store the raw datasets or the full merged dataset in git —
both are reproducible, so committing them would only bloat the repo and force a
Git LFS quota for no benefit.

**1. Download the two raw files** and place them in `data/raw/` (create the folder
if it doesn't exist), keeping these exact names:

| File | Source |
|---|---|
| `Online Retail.xlsx` | https://archive.ics.uci.edu/dataset/352/online+retail |
| `online_retail_II.xlsx` | https://archive.ics.uci.edu/dataset/502/online+retail+ii |

Both are published by the UCI Machine Learning Repository under CC BY 4.0, which
permits use for study and research.

**2. Install Python:** https://www.python.org/ftp/python/pymanager/python-manager-26.3.msix

**3. Remember:** Check the "Add python.exe to PATH" box during the installation process.

**4. Change directory in Terminal** to the folder where you save the notebook.

**5. Then type:** `pip install -r requirements.txt`.

**6. Run the notebook top to bottom.** It will regenerate, locally:
- `data/processed/working_dataset.csv` — the full cleaned/merged dataset
- `data/processed/data_profile.csv` — committed to the repo
- `data/processed/cleaning_log.csv` — committed to the repo
- `data/processed/working_dataset_sample.csv` — a 2,000-row sample, committed to
  the repo so a reader can inspect the schema, including the derived `Category`,
  without downloading anything

`data/raw/` and `working_dataset.csv` are listed in `.gitignore` so they're
never accidentally committed. `RANDOM_SEED = 42` is fixed at the top of the
notebook, so re-running it reproduces the same sample and the same numbers
reported in the analysis.

**7. Output appears in:** `data/processed/`: `working_dataset.csv`, `data_profile.csv`, `cleaning_log.csv`.

# D3 – Data Analysis: Return Rates by Product Category

## 1. Overview
This notebook (`D3_Topic12_DangUyenThu_TruongThuanTuan_NguyenAnKhoi_NguyenThiKhanhLinh_NguyenChiPhuc_DangLeTrucLinh.ipynb`) contains the core data analysis for our project. The objective is to answer the decision question: **"Which product categories generate the most returns, and is it worth acting on?"** 

The analysis is performed by the **Data Analyst (Nguyen An Khoi)**. It covers the analytical pipeline from loading the pre-processed data, executing strict cleaning protocols (handling extreme outliers), calculating descriptive statistics (Return Rate by Line, Return Rate by Value, Excess Return Value), performing a Chi-square test of independence, and finally exporting the processed results for the Visualisation Lead.

## 2. Prerequisites & Environment Setup
To run this notebook successfully on your local machine, you must have Python 3 installed. 

**Step 2.1: Clone the Repository**
Open your terminal (Command Prompt / PowerShell / Terminal) and clone the repository:
```bash
git clone https://github.com/truongtuan1312/introds-project
cd introds-project
```

**Step 2.2: Install Required Libraries**
It is highly recommended to use a virtual environment. To install all required dependencies, run:
```pip install -r requirements.txt```

## 3. Step-by-Step Data Preparation (Crucial Step)

### Why is this step necessary?
The raw datasets are intentionally excluded from this Git repository (via `.gitignore`) because the original Excel files exceed 100MB. Uploading them would violate GitHub's file size limits. Furthermore, Excel files are extremely slow to read. Therefore, we convert them to CSV format for fast, reproducible execution.

### Step 3.1: Download the Raw Data
Download the two original datasets directly from the UCI Machine Learning Repository. You do not need to register; just click download:
* **Source 1:** [Online Retail (Dataset 352)](https://archive.ics.uci.edu/dataset/352/online+retail) $\rightarrow$ Download the `.xlsx` file.
* **Source 2:** [Online Retail II (Dataset 502)](https://archive.ics.uci.edu/dataset/502/online+retail+ii) $\rightarrow$ Download the `.xlsx` file.

### Step 3.2: Convert Excel to CSV
The notebook `D3_Topic12_DangUyenThu_TruongThuanTuan_NguyenAnKhoi_NguyenThiKhanhLinh_NguyenChiPhuc_DangLeTrucLinh.ipynb` reads CSV files, not Excel files. You need to convert them:
1. Open the downloaded `Online Retail.xlsx` file using Microsoft Excel (or Google Sheets).
2. Go to **File** $\rightarrow$ **Save As** (or **Export**).
3. Choose the format: `CSV (Comma delimited) (*.csv)`.
4. Save the file.
5. Repeat this exact process for the `online_retail_II.xlsx` file.

### Step 3.3: Rename and Move to the Correct Folder
The notebook is programmed to look for specific filenames in a specific directory:
1. **Rename the CSV files** you just created exactly as follows (all lowercase, no spaces):
   * Rename the CSV of Source 1 to: `retail1.csv`
   * Rename the CSV of Source 2 to: `retail2.csv`
2. **Create the folder structure:** Inside the root folder of your project (`introds-project`), navigate to the `data` folder. If there is no `raw` folder inside `data`, create one.
3. **Move the files:** Place both `retail1.csv` and `retail2.csv` directly inside the `data/raw/` directory.

Your directory structure **MUST** look exactly like this before proceeding:

```text
introds-project/
├── data/
│   ├── raw/
│   │   ├── retail1.csv    <-- YOU MUST PLACE THIS HERE
│   │   └── retail2.csv    <-- YOU MUST PLACE THIS HERE
│   └── processed/         <-- Notebook will save outputs here automatically
├── notebooks/
│   └── D3_Topic12_DangUyenThu_TruongThuanTuan_NguyenAnKhoi_NguyenThiKhanhLinh_NguyenChiPhuc_DangLeTrucLinh.ipynb  <-- YOU WILL RUN THIS FILE
├── requirements.txt
└── README.md
```
## 4. How to Execute the Notebook

### Step 4.1: Open the Notebook
1. Open the project folder in VS Code or launch Jupyter Notebook.
2. Navigate to `notebooks/D3_Topic12_DangUyenThu_TruongThuanTuan_NguyenAnKhoi_NguyenThiKhanhLinh_NguyenChiPhuc_DangLeTrucLinh.ipynb` and open it.

### Step 4.2: Select the Kernel
If using VS Code, click on the **"Select Kernel"** button in the top right corner and choose your installed Python 3 environment.

### Step 4.3: Run All Cells
1. Click the **"Restart"** button (circular arrow icon) to clear any previous memory.
2. Click the **"Run All"** button (double play icon).
3. Wait for all cells to execute sequentially. The data loading and cleaning process may take 10–20 seconds.

> **Reproducibility Note:**
> The notebook uses a fixed random seed (`RANDOM_SEED = 42`). Running this notebook on any machine will yield the exact same numerical results reported in our final D4 submission.

## 5. Methodology & Cleaning Decisions
To ensure our findings are accurate, the notebook applies strict logic that differs slightly from the raw D2 output:
* **Removing Non-Products:** We drop rows where `StockCode` is not a 5-digit number (removes postage, manual adjustments, discounts).
* **Outlier Handling:** We remove exactly 15 lines with an absolute Quantity $\ge 10,000$. The raw data contains extreme data-entry errors (e.g., two bulk orders of 80,995 and 74,215 units that were cancelled immediately). If kept, these two lines alone would artificially inflate the return rate of their categories by over 10%. A sensitivity check confirms our top categories remain stable across different thresholds.
* **Cancellation Proxy:** As the dataset lacks a literal "return" field, we flag `InvoiceNo` starting with 'C' as a cancellation/return proxy.
* **Primary Metric:** We evaluate returns using **Excess Return Value** (the financial loss exceeding the system-wide average of 2.43%), which directly answers *"is it worth acting on"*.

## 6. Expected Outputs
If the notebook runs successfully, it will automatically generate two files inside the `data/processed/` folder. These files are the handover to the Visualisation Lead for chart generation:
1. `category_summary.csv`: Contains descriptive stats (sales, returns, return rates, excess return values).
2. `category_return_rate_ci.csv`: Contains the 95% Wilson Confidence Intervals for line return rates.

> **Transparency Note:** At the very bottom of the notebook, there is an Appendix cell that explicitly displays the 15 removed outlier rows for transparency.

## 7. Troubleshooting (Common Errors)

* **Error:** `FileNotFoundError: [Errno 2] No such file or directory: '../data/raw/retail1.csv'`
  * **Fix:** You did not place `retail1.csv` and `retail2.csv` in the correct folder, or you named them incorrectly. Re-check Section 3.3.
* **Error:** `ModuleNotFoundError: No module named 'scipy'` (or `statsmodels`)
  * **Fix:** You haven't installed the required libraries. Run `pip install scipy statsmodels` in your terminal.
* **Error:** `ParserError: Error tokenizing data`
  * **Fix:** When saving the CSV from Excel, ensure you used the standard **"CSV (Comma delimited)"** format, not "CSV UTF-8" or "Macintosh".


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
