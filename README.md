# Post-Merger Data Unification Pipeline

A Databricks-based data engineering project that builds a unified, reliable analytics layer following a corporate acquisition — consolidating fragmented data from a parent FMCG company and a newly acquired sportsbar chain into a single source of truth.

---

## Problem Statement

The acquired sportsbar company maintained data across hastily built, inconsistent APIs, leading to widespread data discrepancies and misaligned metrics. This project builds a scalable data foundation that:

- Resolves schema and format inconsistencies between two companies
- Fills missing data and removes duplicates
- Delivers aggregated analytics for both companies in a single dashboard

---

## Architecture Overview

The pipeline follows a **Medallion Architecture** (Bronze → Silver → Gold) built on **Databricks** with **AWS S3** as the source for the child company's data.

| Layer | Scope | Purpose |
|---|---|---|
| Bronze | Child company (sportsbar) | Raw ingestion from S3, metadata added |
| Silver | Child company (sportsbar) | Cleaned, standardised, schema-aligned |
| Gold | Both companies combined | Merged, analytics-ready, upserted |

Data modeling follows a **Star Schema** with a central fact table (orders) surrounded by dimension tables (customers, products, pricing, dates).

---

## Tech Stack

- **Platform**: Databricks (Unity Catalog, Workflows, Genie AI)
- **Storage**: AWS S3 (child company source data), Databricks Volumes (parent company)
- **Processing**: PySpark, Delta Lake, SQL
- **Orchestration**: Databricks Jobs (DAG-based task dependencies)
- **Dashboarding**: Databricks Dashboard + Genie (natural language querying)
- **Data Modeling**: Star Schema

---

## Dataset from Kaggle

Public dataset covering two companies with the following entities:

- **Customers** — brand, demographics, identifiers
- **Products** — product catalog, categories
- **Pricing** — pricing tables per product
- **Orders** — transactional fact data (historical and incremental)

---

## Pipeline Structure

```
consolidated_pipeline/
├── setup/
│   ├── setup.py               # Catalog and schema creation
│   └── utilities.py           # Shared config (catalog name, schema names)
├── dimension_data_processing/
│   ├── customer_dim.py        # Customers: Bronze → Silver → Gold merge
│   ├── products_dim.py        # Products: Bronze → Silver → Gold merge
│   └── pricing_dim.py         # Pricing: Bronze → Silver → Gold merge
├── fact_data_processing/
│   ├── historical_load.py     # One-time backfill of orders
│   └── incremental_load.py    # Daily incremental orders via staging tables
└── orchestration/
    └── job_config.md          # Databricks Jobs DAG definition
```

---

## Key Design Decisions

**Historical backfill + incremental load separation**
The pipeline handles two distinct load patterns. A one-time historical load ingests all past orders. Going forward, daily incremental files land in S3 and are processed through temporary staging tables in Bronze and Silver, then merged into Gold and dropped — keeping the pipeline clean and storage-efficient.

**Staging table pattern for incremental data**
Incremental orders are loaded into a staging table rather than directly into Gold. This isolates new data, allows validation before merging, and avoids partial writes to the production Gold layer.

**Parameterised notebooks with widgets**
All notebooks accept `catalog` and `data_source` as configurable parameters via Databricks widgets, making them reusable across environments without code changes.

**Upsert (MERGE) operations at Gold layer**
All Gold layer writes use Delta Lake MERGE to handle deduplication and idempotency — safe to re-run without creating duplicate records.

---

## Orchestration

Databricks Jobs runs the full pipeline daily in dependency order:

```
dim_processing_customers
        ↓
dim_processing_products
        ↓
dim_processing_pricing
        ↓
fact_processing_orders
```

Each task is parameterised with `catalog` and `data_source` values, making the DAG fully config-driven.

---

## Dashboard and AI Querying

A denormalised view joins the Gold fact and dimension tables, powering:

- **Databricks Dashboard** — combined KPIs, order trends, product performance across both companies
- **Databricks Genie** — natural language querying over the unified Gold layer
- 

---

## How to Run

1. Set up a Databricks workspace with Unity Catalog enabled
2. Create an external location pointing to your S3 bucket (child company data)
3. Run `setup/setup.py` to initialise catalog and schemas
4. Upload parent company data to Databricks Volumes
5. Upload child company data to S3
6. Run dimension notebooks in order, then fact notebooks
7. For ongoing use, configure the Databricks Job for daily incremental runs

---

## Author

Sravya Velgapuri
[LinkedIn](www.linkedin.com/in/sravya-velgapuri9) | [GitHub](https://github.com/Sravyaraov)
