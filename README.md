# Data Ingestion, Cleaning & Preprocessing with Pandas — E-Commerce Transactions

Internship task: take a messy, 10,000+ row real-world business dataset and clean it thoroughly,
with a notebook showing the before/after state.

## What's in this repo

| Path | What it is |
|---|---|
| `data/ecommerce_transactions_raw.csv` | Raw dataset — 10,710 e-commerce order transactions, intentionally messy |
| `data/clean_dataset.csv` | Final cleaned, standardized output (10,503 rows) — the deliverable |
| `data/before_cleaning_outliers.png`, `data/after_cleaning_outliers.png` | Boxplot/histogram snapshots referenced in the notebook |
| `notebooks/Ecommerce_Data_Cleaning.ipynb` | Full, already-executed cleaning notebook |

## Dataset

One row = one order transaction: `transaction_id`, `order_date`, `customer_id`,
`customer_name`, `product_category`, `product_name`, `quantity`, `unit_price`, `unit_cost`,
`discount_pct`, `payment_method`, `region`, `order_status`.

## Issues found and fixed (see the notebook for full detail on each)

- **Missing values** in `customer_name`, `discount_pct`, `region`, `payment_method`,
  `unit_cost`, `order_status` — imputed per-column with a documented, field-appropriate
  strategy (mode, 0, category-wise median, or an explicit "Unknown" label); rows missing a
  core field (`order_date`, `quantity`, `unit_price`) were dropped since there's no safe way
  to guess those.
- **Duplicate rows** — exact duplicates removed, keeping the first occurrence.
- **Inconsistent data types** — `quantity` had number-words ("two"), `unit_price` had a ₹
  currency symbol and thousands separators; both coerced to proper numeric types.
- **Mixed date formats** — `order_date` mixed ISO (`YYYY-MM-DD`) and `DD/MM/YYYY`; parsed to a
  single datetime type.
- **Inconsistent casing** — `product_category` and `order_status` had mixed
  UPPER/lower/Title case; standardized to Title Case.
- **Outliers** — extreme `quantity` (500–999) and `unit_price` (₹500K+) values capped using
  the IQR method rather than dropped, to preserve the transaction while preventing distortion.
- **Negative values** — a handful of negative `unit_cost` entries (sign errors) corrected via
  absolute value.

## Feature engineering

- `order_year`, `order_month`, `order_month_name` — extracted from `order_date`
- `revenue` = quantity × unit_price × (1 − discount_pct/100)
- `total_cost` = quantity × unit_cost
- `profit` = revenue − total_cost
- `profit_margin_pct` = (profit / revenue) × 100

## How to reproduce

```bash
pip install pandas numpy matplotlib jupyter
jupyter nbconvert --to notebook --execute notebooks/Ecommerce_Data_Cleaning.ipynb
```

This re-reads `data/ecommerce_transactions_raw.csv`, re-runs every cleaning step, and rewrites
`data/clean_dataset.csv`.

## Before vs. after (last run)

| Metric | Before | After |
|---|---|---|
| Rows | 10,710 | 10,503 |
| Exact duplicate rows | ~210 | 0 |
| Total missing values | ~1,600 | 0 |
| Negative `unit_cost` rows | 15 | 0 |
| `product_category` distinct spellings | 18 (casing variants) | 6 (canonical) |
| `order_date` formats | 2 (ISO + DD/MM/YYYY) | 1 (datetime) |

Exact counts are in the notebook's Section 5 "Before vs. After Summary" cell, since they can
shift slightly between re-runs of the random raw-data generator.
