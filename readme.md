# GCP Data Engineering: End-to-End Olist ELT Pipeline

This project demonstrates a professional Cloud Data Engineering workflow using **Google Cloud Platform**. Unlike traditional local analysis, this architecture focuses on scalability, automated transformations via **Dataform**, and cost-optimization in **BigQuery**.

![Project Architecture](./images/Arborescence_projet_gcp.drawio.png)

## 🎯 Project Overview
The goal was to move beyond static analysis to create a production-ready data pipeline. I transformed raw Brazilian e-commerce data (Olist) into a clean, optimized analytical warehouse to drive business KPIs like Revenue Trends and Product Performance.

## 🏗️ Architecture & Tech Stack
* **Ingestion:** CSV data hosted on **Google Cloud Storage**.
* **Data Warehouse:** **Google BigQuery** (Separated into `raw_data` and `analytics` datasets).
* **Transformation:** **Dataform (SQLX)** for modular ELT, dependency management, and data quality.
* **Optimization:** Table Partitioning and Clustering for cost-efficient querying.
* **Visualization:** **Looker Studio** for executive-level reporting.

## 🛠️ Dataform Workflow & Lineage
The pipeline follows a modular approach to ensure data integrity and reusability:

1.  **Staging Layer:** Cleaned raw strings, cast data types (Prices to `FLOAT64`, Dates to `TIMESTAMP`), and standardized column names.
2.  **Analytics Layer:** Joined multiple sources (Orders, Items, Products) into a centralized `fact_order_items` table.
3.  **Documentation:** Integrated descriptions and assertions directly into the SQLX definitions.

![Dataform Compiled Graph](./images/dataform-compiled-graph.png)

## ⚡ Performance Optimizations
To ensure the dashboard remains fast and BigQuery costs remain low:
* **Partitioning:** The `fact_order_items` table is partitioned by `purchase_at` (Day), allowing Looker Studio to scan only relevant date ranges.
* **Clustering:** Data is clustered by `product_category_name`, significantly speeding up category-specific filtering.

## 📊 Business Insights
The final dashboard provides a real-time view of:
* **Financials:** Total Revenue, Average Order Value (AOV), and Sales Volume.
* **Trends:** Monthly revenue evolution tracking growth patterns.
* **Category Analysis:** Identification of top-performing product segments.

![Looker Studio Dashboard](./images/Olist_Performance_Looker_Dashboard.png)

## 🛠️ Troubleshooting & Technical Challenges

Building this pipeline involved solving several real-world data engineering hurdles:

### 1. Data Type Mismatch (The "String" Trap)
* **Issue:** During the initial ingestion from CSV to BigQuery, all columns (including prices and dates) were imported as `STRING`. This prevented any mathematical aggregation or time-series analysis.
* **Solution:** In the Dataform staging layer (`stg_orders`, `stg_items`), I implemented explicit `CAST` functions:
    * `CAST(price AS FLOAT64)`
    * `CAST(purchase_at AS TIMESTAMP)`
* **Impact:** Enabled the "Average Order Value" calculation and the "Revenue Over Time" trend chart.

### 2. Dashboard Aesthetic & Metric Accuracy
* **Issue:** Initial scorecards showed confusing decimal places (e.g., "271.0" orders) and lacked currency context, making the dashboard look unprofessional.
* **Solution:** * Adjusted **Decimal Precision** to `0` for count-based metrics.
    * Applied **Currency Formatting (BRL)** to revenue metrics.
    * Renamed technical column names (e.g., `product_category_name`) to user-friendly labels (e.g., "Product Category").

### 3. Dataform Dependency Management
* **Issue:** When running the execution, some tables failed because they tried to populate before their source tables were ready.
* **Solution:** Used the `ref()` function in SQLX to define a strict DAG (Directed Acyclic Graph). This ensured the `fact_order_items` table only executed after `stg_orders`, `stg_items`, and `stg_products` were successfully updated.

### 4. BigQuery Cost & Performance Optimization
* **Issue:** Standard queries on large datasets can be expensive and slow if they scan the whole table every time.
* **Solution:** Implemented **Partitioning** on the `purchase_at` column. This forces BigQuery to only scan the specific data "shards" requested by the Looker Studio date filter, drastically reducing costs and query time.

---
**Note:** This is part of a series of projects using the Olist dataset, focusing specifically on **Cloud Infrastructure** and **Modern Data Stack (MDS)** tools.