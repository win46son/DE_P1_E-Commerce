# Enterprise E-Commerce Data Lakehouse (Databricks & Unity Catalog)

A production-grade, end-to-end Data Lakehouse architecture built on Databricks using PySpark, Delta Lake, and Unity Catalog, implementing the Medallion Architecture (Bronze, Silver, Gold).

---

## 🏗️ System Architecture & Medallion Design

This project simulates a modern enterprise e-commerce platform ingesting multi-source heterogeneous raw data (CRM, ERP, Transactional Orders, and Clickstream events) and processing it through a three-layer Medallion architecture.

```
+-----------------------------------------------------------------------------------+
|                                  RAW DATA SOURCES                                 |
|         (Parquet / JSON Files Ingested into Unity Catalog Volume)                |
+-----------------------------------------------------------------------------------+
                                          |
                                          v
+-----------------------------------------------------------------------------------+
|                                  BRONZE LAYER                                     |
|  - Raw Data Landing (As-Is Ingestion)                                            |
|  - Metadata Enrichment (ingestion_timestamp, source_system)                       |
|  - Persistence: Unity Catalog Delta Managed Tables                                |
+-----------------------------------------------------------------------------------+
                                          |
                                          v
+-----------------------------------------------------------------------------------+
|                                  SILVER LAYER                                     |
|  - Data Cleaning & Schema Standardization                                         |
|  - Quarantine Pattern: Isolate invalid/corrupted records                          |
|  - SCD Type 2 (Slowly Changing Dimension): Historical user tracking              |
+-----------------------------------------------------------------------------------+
                                          |
                                          v
+-----------------------------------------------------------------------------------+
|                                   GOLD LAYER                                      |
|  - Star Schema Dimensional Modeling                                               |
|  - Wide Fact Table (gold_order_details_fact)                                      |
|  - Aggregated Metrics (gold_daily_sales_summary: GMV, Orders, AOV)                |
+-----------------------------------------------------------------------------------+
```

---

## 🛠️ Key Technical Features & Engineering Patterns

1. **Unity Catalog (UC) Governance Integration**
   - Utilizes UC Volumes (`/Volumes/main/ecommerce/raw_volume`) for structured unstructured/raw file landing.
   - Enforces 3-tier namespace catalog governance (`main.ecommerce.table_name`) across all Managed Delta Tables.

2. **Quarantine Pattern (Data Quality Control)**
   - Protects downstream main analytical pipelines by detecting data quality violations (null keys, negative quantities, cancelled/refunded status).
   - Routes non-compliant data into `quarantine_orders` with audit reason codes for root-cause analysis without breaking pipeline execution.

3. **SCD Type 2 Dimension Tracking (Slowly Changing Dimensions)**
   - Manages mutable user profile attributes (e.g., membership tier changes, city updates) using validity intervals (`start_date`, `end_date`) and status flags (`is_current`).
   - Ensures **point-in-time correctness** for historical reporting.

4. **Star Schema & Analytical Pre-Aggregation**
   - Denormalizes Silver layer entities into enterprise fact tables (`gold_order_details_fact`).
   - Pre-aggregates high-frequency business metrics (Daily GMV, Category Performance, Average Order Value) for BI performance optimization.

---

## 📊 Medallion Layer Specifications

| Layer | Table Name | Description | Key Transformations |
| :--- | :--- | :--- | :--- |
| **Raw Volume** | `raw_volume` | Cloud Object Storage Landing Zone | Parquet / JSON raw multi-source files |
| **Bronze** | `bronze_users` | CRM User Profile Snapshot | Audit fields added (`ingestion_timestamp`, `source_system`) |
| **Bronze** | `bronze_products` | ERP Product Metadata | Audit fields added (`ingestion_timestamp`, `source_system`) |
| **Bronze** | `bronze_orders` | Transactional Order Stream | Partitioned by date (`source_file_date`) |
| **Bronze** | `bronze_events` | Clickstream JSON Logs | Semi-structured schema parsing |
| **Silver** | `silver_orders` | Cleaned Orders Fact | Schema standardization, valid order filtering |
| **Silver** | `quarantine_orders` | Isolated Exception Table | Captures null keys, bad metrics, and cancelled orders |
| **Silver** | `silver_users_scd2` | User Dimension Table (SCD2) | Manages historical state changes over time |
| **Gold** | `gold_order_details_fact` | Star Schema Fact Table | Joins Silver Orders, Products, and Current Users |
| **Gold** | `gold_daily_sales_summary` | Business Aggregation Table | GMV, Order Counts, AOV aggregated by date & tier |

---

## 🚀 Tech Stack

- **Platform:** Databricks (Community Edition / Enterprise)
- **Data Governance:** Unity Catalog (UC Volumes & Managed Delta Tables)
- **Engine:** Apache Spark / PySpark SQL
- **Storage Format:** Delta Lake (ACID Transactions, Time Travel, Schema Evolution)
- **Language:** Python 3.10+, PySpark, SQL