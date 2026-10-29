# Databricks Medallion Architecture — Mini Project

A hands-on tutorial series that teaches the **medallion architecture** on Databricks. It can be performed on the Databricks Free Edition — no cloud storage or paid compute required — and is a great starting point for understanding how data flows through the medallion layers on Databricks and the cloud.

## What is the medallion architecture?

The medallion architecture is Databricks' recommended way to organize a lakehouse: data flows through three layers, each more refined than the last.

* **Bronze** — raw data landed **as-is**, exactly as it arrived. No cleaning, no reshaping. It is the immutable source of truth for reprocessing.
* **Silver** — cleaned, validated, and standardized data: duplicates removed, bad values filtered, text standardized, useful columns added.
* **Gold** — aggregated, business-ready data, designed to answer specific business questions and power dashboards and reports.

## The journey in this project

```
Data/Products.csv  (raw sales invoices)
      │
      │  Notebook 01 — read the CSV into a Spark DataFrame and explore it
      │  Notebook 02 — land the raw data as-is
      ▼
demo.demo_sch.products_bronze            (Bronze — raw, 2,000 rows)
      │
      │  Notebook 03 — data quality checks, dedupe, clean, standardize, enrich
      ▼
demo.demo_sch.products_silver            (Silver — clean, one row per invoice)
      │
      │  Notebook 04 — aggregate to a business grain
      ▼
demo.demo_sch.gold_country_monthly_sales  (Gold — one row per country-month)
```

## Notebooks

| # | Notebook | What it teaches | What it produces |
|---|---|---|---|
| 01 | Read and Explore Data with Spark | Explicit schemas (vs `inferSchema`), reading workspace files into a Spark DataFrame, lazy evaluation, exploration aggregations, NULL checks | Nothing persistent — exploration only |
| 02 | Save Data to the Bronze Layer | What Bronze is for, writing a Delta table with `saveAsTable`, overwrite mode, read-back verification | `demo.demo_sch.products_bronze` |
| 03 | Clean Data into the Silver Layer | Data quality checks (duplicates, unexpected values), defensive cleaning (`dropDuplicates`, validity filters, `trim`/`initcap`), enrichment | `demo.demo_sch.products_silver` |
| 04 | Aggregate Data into the Gold Layer | Designing a Gold table from a business question, grain, grouped vs aggregated columns, window functions | `demo.demo_sch.gold_country_monthly_sales` |

Each notebook starts with a **Roadmap** markdown cell and ends with a **Recap**, so it works as standalone teaching material.

## The dataset

`Data/Products.csv` — 2,000 sales invoices from a bike and accessories shop:

| Column | Description |
|---|---|
| `OrderDate` | Order date (dd-MM-yyyy, 2021-01-01 → 2024-03-26) |
| `InvoiceNum` | Unique invoice number — the natural key |
| `ProductKey` / `EnglishProductName` | Product identifier and name |
| `Country` | One of 7 countries (Australia, Canada, France, Germany, India, Italy, United States) |
| `SalesAmount` / `UnitPrice` / `OrderQuantity` | Sale value, unit price, quantity |
| `TaxAmt` | Tax (consistently 8% of the sale) |
| `TotalProductCost` | Cost of goods sold |

## The three layers, side by side

| Layer | Table | Rows | Grain (what each row is) |
|---|---|---|---|
| Bronze | `demo.demo_sch.products_bronze` | 2,000 | One raw invoice, as ingested |
| Silver | `demo.demo_sch.products_silver` | 2,000 | One validated invoice, enriched with `SalesYear`, `SalesMonth`, `ProfitAmount` |
| Gold | `demo.demo_sch.gold_country_monthly_sales` | 261 | One country's sales for one month — invoice count, units, sales/tax/cost/profit, average order value, share of monthly revenue |

## How to run

1. Run the notebooks **in order** (01 → 04) on any Databricks compute (serverless works out of the box).
2. Notebooks 02–04 each re-create what they need, so they also run standalone.
3. All writes use `mode("overwrite")` — every notebook is safely re-runnable.
4. The Unity Catalog schema used is `demo.demo_sch`; adjust table names there if your workspace differs.

## Concepts covered

* Reading files into Spark DataFrames with explicit schemas
* Delta tables, `saveAsTable`, and overwrite semantics
* Data quality: duplicate detection, unexpected-value checks, NULL scans
* Cleaning and standardization: `dropDuplicates`, filters, `trim`/`initcap`
* Enrichment: derived columns and date parts
* Designing a Gold table: business question → grain → grouped vs aggregated columns
* Aggregations, window functions, and verifying a table's grain
