# Vendor Performance Analysis (Python + Power BI)

> End-to-end vendor and inventory performance analytics with automated ingestion, KPI generation, and an interactive Power BI dashboard.

<img width="1803" height="1004" alt="Workflow" src="https://github.com/user-attachments/assets/e9c8d2e5-a08a-4187-9aab-6bbe931724b5" />


---

## 🚀 Overview

This project analyzes vendor performance for a wholesale/retail business to surface profit drivers and operational issues (pricing mismatches, stuck inventory, margin leakage). It delivers:

* Automated ingestion + cleaning of raw CSVs into a single SQLite database
* Reproducible EDA and vendor/brand performance analysis (Jupyter + Pandas/SQL)
* Python automation to generate vendor-level summary reports
* A business-friendly, interactive Power BI dashboard
* A final PDF report with actionable insights

## 🧰 Tech Stack

* **Python**: pandas, matplotlib, sqlite3 (optional: numpy, logging)
* **Notebooks**: Jupyter
* **BI**: Power BI (.pbix)
* **Storage**: SQLite
* **Other**: Excel/CSV, PDF reporting

## 📦 Project Structure

```
├── data/
│   └── vendor_sales_summary.csv
├── notebooks/
│   ├── Ingestion.ipynb
│   ├── Exploratory Data Analysis.ipynb
│   └── Vendor Performance Analysis.ipynb
├── scripts/
│   ├── ingestion_db.py
│   └── get_vendor_summary.py
├── db/
│   └── inventory.db
├── dashboard/
│   └── vendor_performance.pbix
├── reports/
│   └── Vendor Performance Report.pdf
├── outputs/
│   ├── final_table.csv
│   └── vendor_summaries/
└── README.md
```

> **Note:** Paths can be customized via environment variables (see `.env.example`).

## 🧩 Data Model (Assumptions)

Core tables are unified into `inventory.db`:

* `sales(product_id, date, qty_sold, sales_amount, vendor_name, brand, store)`
* `purchases(product_id, po_date, qty_purchased, unit_cost, vendor_name)`
* `inventory(product_id, opening_qty, closing_qty, opening_value, closing_value)`
* `products(product_id, sku, product_name, category, brand, vendor_name)`

> Adjust queries in notebooks/scripts if your schema differs.

## 🔑 KPIs & Diagnostics

* **Revenue**, **COGS**, **Gross Margin** (absolute & %)
* **Sell-through** %, **Days on Hand**, **Aging Buckets**
* **Price Realization vs List/Contract**
* **Bulk Purchase Impact**: volume discount vs margin impact
* **Slow/Stuck Inventory**: low velocity & high age flags

## 🗺️ Workflow

1. **Ingest** → Load CSVs into SQLite with logging & type coercion
2. **Clean** → Deduplicate, normalize dates, handle missing values
3. **Model** → Join sales/purchases/inventory into an analysis-ready fact table
4. **Analyze** → Compute KPIs and vendor/brand-level stats
5. **Visualize** → Power BI dashboard binds to `outputs/final_table.csv`
6. **Report** → Export insights to `reports/Vendor Performance Report.pdf`

## ⚙️ Quickstart

```bash
# 1) Clone
git clone <your-repo-url>
cd vendor-performance

# 2) Create env
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
pip install -r requirements.txt

# 3) Configure (optional)
cp .env.example .env  # edit paths if needed

# 4) Ingest to SQLite
python scripts/ingestion_db.py --data-dir ./data --db ./db/inventory.db

# 5) Generate analysis table for BI
python scripts/get_vendor_summary.py \
  --db ./db/inventory.db \
  --out ./outputs/final_table.csv \
  --summaries ./outputs/vendor_summaries

# 6) Open Dashboard
# Open dashboard/vendor_performance.pbix and point to outputs/final_table.csv
```

## 🐍 Example Snippets

**Ingestion (to SQLite)**

```python
import pandas as pd, sqlite3, pathlib
conn = sqlite3.connect('db/inventory.db')
for csv in pathlib.Path('data').glob('*.csv'):
    df = pd.read_csv(csv)
    df.to_sql(csv.stem, conn, if_exists='replace', index=False)
```

**Cleaning & Merge**

```python
import pandas as pd, sqlite3
conn = sqlite3.connect('db/inventory.db')
sales = pd.read_sql('select * from sales', conn)
purch = pd.read_sql('select * from purchases', conn)
merged = sales.merge(purch, on='product_id', how='left', suffixes=('', '_po'))
merged['date'] = pd.to_datetime(merged['date'])
merged.dropna(subset=['sales_amount'], inplace=True)
merged.to_csv('outputs/final_table.csv', index=False)
```

**Vendor Ranking**

```python
vendor_sales = merged.groupby('vendor_name', as_index=False)['sales_amount'].sum()
top5 = vendor_sales.sort_values('sales_amount', ascending=False).head(5)
```

## 📊 Power BI Dashboard

* Pages: **Overview, Vendor Health, Inventory Aging, Pricing, Margin Bridge**
* Filters: date range, vendor, brand, category, store
* Exports: PDF/PNG; drill-through to vendor detail

Add images to `assets/` for README previews:

* `assets/Dashboard.gif` – animated overview
* `assets/Dashboard_Vendor.png` – vendor detail

## 🧪 Validation

* Row-count & schema checks post-ingestion
* Null/duplicate audits
* Reconciliation: Sales vs Inventory movement
* Sanity checks for margin (>=0 unless returns/markdowns)

## 📝 Reporting

`reports/Vendor Performance Report.pdf` summarizes:

* Key wins + risks by vendor/brand
* Margin drivers + leakage
* Stuck inventory and liquidation candidates
* Pricing actions & reorder guidance

## 📈 Outputs

* `outputs/final_table.csv` – clean, analysis-ready
* `outputs/vendor_summaries/*.csv` – per-vendor KPIs
* `dashboard/vendor_performance.pbix` – interactive BI
* `reports/Vendor Performance Report.pdf` – exec-ready insights

## 🗺️ Roadmap

* [ ] Incremental ingestion + change-data-capture
* [ ] dbt models for transform lineage
* [ ] CI data tests (Great Expectations)
* [ ] Parameterized CLI for date/vendor scopes
* [ ] Optional cloud warehouse target (DuckDB/BigQuery/Snowflake)

## 🤝 Contributing

PRs are welcome! For major changes, open an Issue first to discuss scope.

## 🙋 Contact

Questions or collaboration? Connect via **LinkedIn** or open a **GitHub Issue**.

## 📌 Sample Dashboard

![Dashboard](https://github.com/user-attachments/assets/5326cea3-b946-4781-8632-cb517a486011)

