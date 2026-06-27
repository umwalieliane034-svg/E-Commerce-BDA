# E-Commerce-BDA
Multi-model e-commerce big data pipeline leveraging MongoDB for transactional master catalogs and HBase for high-velocity time-series clickstream logs. Includes automated data ingestion, advanced ANSI SQL analytical integration queries, and a custom data visualization suite for end-to-end conversion funnel metrics.

# Multi-Model E-Commerce Big Data Architecture & Analytics Pipeline

This repository contains the complete implementation of a production-grade big data architecture designed to ingest, clean, and analyze enterprise-scale e-commerce datasets. The system leverages a multi-model database strategy using **MongoDB** for transactional entities and **HBase** for time-series clickstream activity logs, with an integrated **ANSI SQL processing tier** for unified analytics.

---

## System Prerequisites & Installation

To reproduce the results locally, ensure the following dependencies are installed:

```bash
# 1. Install required Python packages via pip
pip install pymongo happybase pandas matplotlib seaborn duckdb
```

Ensure **MongoDB Server**, **HBase Standalone** (via Docker or local binary paths), and your preferred Python IDE (or Jupyter Notebook environment) are active.

---

##  Step-by-Step Reproduction Guide

### Step 1: Population of the MongoDB Core Layer
Run the database loader script to cleanly initialize the collections (`categories`, `users`, `products`, `transactions`) and populate them directly from the source files:

```bash
python populate_mongodb.py
```

### Step 2: Initialize Columns and Ingest HBase Records
1. Enter active HBase shell (e.g., `docker exec -it hbase-standalone hbase shell`) and run:
   ```hbase
   create 'sessions', 'cf_meta', 'cf_geo', 'cf_activity'
   create 'product_metrics', 'cf_stats'
   ```
2. Execute the Python loader script to inject the time-series activity telemetry tracks:
   ```bash
   python populate_hbase.py
   ```

### Step 3: Execute the Data Processing and Integration Pipeline
Execute the analytical framework to perform data cleaning, handle structural updates, map payment vector cohorts, and run cross-store SQL inner joins matching session logs against core checkout tables:

```bash
python spark_sql_analytics.py
```

### Step 4: Generate the Corporate Visualization Charts
Generate the static enterprise charts mapping distribution data and customer behavior trends:

```bash
python visualization_suite.py
```

---

##  Sample Queries & Verified Reference Outputs

### Query 1: MongoDB Non-Trivial Aggregation (Category Sales)
```python
# To verify this query, execute it inside MongoDB Compass or a Python shell
pipeline = [
    { "$unwind": "$items" },
    { "$lookup": { "from": "products", "localField": "items.product_id", "foreignField": "_id", "as": "p" } },
    { "$unwind": "$p" },
    { "$lookup": { "from": "categories", "localField": "p.category_id", "foreignField": "_id", "as": "c" } },
    { "$unwind": "$c" },
    { "$group": { "_id": "$c.name", "revenue": { "$sum": "$items.subtotal" }, "sold": { "$sum": "$items.quantity" } } },
    { "$sort": { "revenue": -1 } }
]
```

### Query 2: HBase User Session Chronological Prefix Scan
```python
# Fetches user activity logs instantly from disk blocks ordered by reverse timestamp
row_prefix = f"{target_user_id}#".encode('utf-8')
for key, data in table.scan(row_prefix=row_prefix):
    print(f"RowKey: {key.decode()} -> Session ID: {data[b'cf_meta:session_id'].decode()}")
```

### Query 3: Multi-Store Cross-Database ANSI SQL Inner Join
```sql
-- Unifies MongoDB transaction tables and HBase clickstream tables over session tokens
SELECT 
    s.session_id, s.user_id, s.device_profile['os'] AS operating_system,
    t.transaction_id, t.total AS purchase_amount, t.payment_method
FROM v_sessions s
INNER JOIN v_transactions_clean t ON s.session_id = t.session_id
LIMIT 5;
```

### Reference Target Result Matrix (Platform Conversion Funnel)
```text
Stage 1 (Awareness - Total Browsed)  : 100,000 unique sessions
Stage 2 (Consideration - Cart Adds)   : 64,939 unique sessions (64.94%)
Stage 3 (Conversion - Completed Txns) : 20,239 unique orders   (31.17%)
Overall E-Commerce Platform Conversion Rate: 20.24%
```

---

## Core System Documentation
The final architectural assessments, detailed structural evaluations, schema decisions, and database comparison matrices can be reviewed directly inside the complete submission **Technical Report document**.





