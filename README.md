# nyc-taxi-etl-pipeline
End-to-End Data Engineering Pipeline using Databricks, PySpark, and Azure Data Lake.
# NYC Taxi ETL Pipeline: End-to-End Medallion Architecture

## 📌 Project Overview
This project is a fully automated, cloud-native Data Engineering pipeline built on **Databricks** and **Azure Data Lake Storage (ADLS Gen2)**. It extracts millions of rows of NYC Taxi data, processes it through a Medallion Architecture (Bronze, Silver, Gold), and serves the aggregated data to a business intelligence dashboard. 

The entire pipeline is governed by **Databricks Unity Catalog** for strict access control and is orchestrated using **Databricks Workflows** for automated monthly runs.

## 🏗️ Architecture & Tech Stack
* **Cloud Provider:** Microsoft Azure (ADLS Gen2)
* **Data Processing:** PySpark, Spark SQL, Databricks
* **Data Governance:** Databricks Unity Catalog (External Locations, Managed Identities)
* **Storage Format:** Delta Lake (Parquet + Transaction Logs)
* **Orchestration:** Databricks Workflows (Jobs)
* **Visualization:** Databricks SQL Dashboards

## ⚙️ Pipeline Stages (Medallion Architecture)

### 1. Raw Data Ingestion (Databricks Volumes)
* Downloads the monthly NYC Yellow Taxi Parquet files from the official TLC website.
* Lands the unstructured data directly into a Unity Catalog **Volume** (`raw_landing`).

### 2. Bronze Layer (Raw Data)
* Reads the raw Parquet files from the Volume.
* Appends an `ingestion_timestamp` for data lineage tracking.
* Saves the data as an **External Delta Table** in a dedicated Azure container.

### 3. Silver Layer (Cleaned & Transformed Data)
* Reads from the Bronze Delta Table.
* Applies data quality filters:
  * Removes trips with `0` or negative distances and passengers.
  * Removes erroneous refund trips (`total_amount < 0`).
  * Validates that dropoff times occur *after* pickup times.
* Calculates derived metrics, such as `trip_duration_minutes`.
* Overwrites the Silver External Delta Table with clean, trustworthy data.

### 4. Gold Layer (Business Aggregates)
* Reads from the Silver Delta Table.
* Aggregates millions of rows into a highly optimized table for business reporting.
* Groups data by `pickup_date` and `pickup_hour` to calculate:
  * `total_trips`
  * `total_revenue`
  * `avg_fare_per_trip`
  * `avg_duration_minutes`

## 📊 Analytics & Dashboards
The pipeline culminates in an automated Databricks SQL Dashboard designed for executive review. Key metrics tracked include:
* **Peak Demand Tracker:** Bar charts highlighting the busiest hours of operation.
* **Revenue Generation:** Trend lines showing revenue growth over time.
* **Efficiency Scorecard:** High-level KPIs tracking the average revenue per trip.
<img width="938" height="729" alt="Screenshot 2026-03-28 1224312345" src="https://github.com/user-attachments/assets/f9ea6c57-19e0-44df-9a12-0a75641e6b68" />

<img width="938" height="729" alt="Screenshot 2026-03-28 1224312345" src="https://github.com/user-attachments/assets/fc257949-ac87-4923-a381-80f8cd5581aa" />



## 🚀 Automation
The pipeline is fully orchestrated using **Databricks Workflows**. A cron-scheduled job runs on the 1st of every month to:
1. Trigger the Raw -> Bronze ingestion.
2. Trigger the Bronze -> Silver cleaning upon successful ingestion.
3. Trigger the Silver -> Gold aggregation.
4. Refresh the Executive Dashboard cache for zero-latency reporting.
