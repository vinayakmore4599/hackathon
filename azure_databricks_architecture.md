# Azure + Databricks Data Architecture for Retail Sales Performance

## Executive Summary
This document outlines a cloud-native data architecture using **Microsoft Azure** and **Databricks** to build a scalable, analytics-ready platform for retail sales performance. The architecture implements the Star Schema data model with modern data lakehouse capabilities.

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           DATA SOURCES (CSV Files)                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│  │   Customer   │  │   Product    │  │    Sales &   │  │   Payment &  │   │
│  │     Data     │  │     Data     │  │    Returns   │  │   Refunds    │   │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘   │
└────────────────────────────────────────┬─────────────────────────────────────┘
                                         │
                ┌────────────────────────┴──────────────────────────┐
                │                                                   │
                ▼                                                   ▼
    ┌──────────────────────────────┐             ┌──────────────────────────────┐
    │  AZURE DATA FACTORY (ADF)    │             │   AZURE EVENT HUBS/          │
    │  - Data Ingestion Pipelines  │             │   AZURE LOGIC APPS           │
    │  - Orchestration             │             │   - Real-time Ingestion      │
    │  - Scheduling                │             │   - Stream Processing        │
    └──────────────┬───────────────┘             └──────────────┬───────────────┘
                  │                                              │
                  └──────────────────────┬─────────────────────┘
                                        │
                        ┌───────────────▼──────────────┐
                        │  AZURE DATA LAKE GEN2 (ADLS2)│
                        │  ┌────────────────────────┐  │
                        │  │  Raw Layer (Bronze)    │  │
                        │  │  - CSV uploads         │  │
                        │  │  - Incremental data    │  │
                        │  └────────────────────────┘  │
                        │  ┌────────────────────────┐  │
                        │  │ Processed Layer (Silver)│  │
                        │  │  - Cleaned data        │  │
                        │  │  - Deduplicated        │  │
                        │  │  - Validated           │  │
                        │  └────────────────────────┘  │
                        │  ┌────────────────────────┐  │
                        │  │ Analytics Layer (Gold) │  │
                        │  │  - Star Schema         │  │
                        │  │  - Aggregates          │  │
                        │  │  - Cubes               │  │
                        │  └────────────────────────┘  │
                        └───────────────┬───────────────┘
                                        │
                    ┌───────────────────┴────────────────────┐
                    │                                        │
                    ▼                                        ▼
        ┌──────────────────────────┐          ┌──────────────────────────┐
        │   DATABRICKS WORKSPACE   │          │  AZURE SQL DATABASE      │
        │  ┌────────────────────┐  │          │  ┌────────────────────┐  │
        │  │ Spark Clusters     │  │          │  │ Data Warehouse     │  │
        │  │ - Transformation   │  │          │  │ - Star Schema      │  │
        │  │ - Processing       │  │          │  │ - Aggregates       │  │
        │  │ - ML Models        │  │          │  │ - Reporting DB     │  │
        │  └────────────────────┘  │          │  └────────────────────┘  │
        │  ┌────────────────────┐  │          │  ┌────────────────────┐  │
        │  │ Delta Lake Tables  │  │          │  │ Synchronized with  │  │
        │  │ - CUSTOMER_DIM     │  │          │  │ Databricks via:    │  │
        │  │ - PRODUCT_DIM      │  │          │  │ - JDBC connector   │  │
        │  │ - SALES_FACT       │  │          │  │ - Spark SQL        │  │
        │  │ - PAYMENT_FACT     │  │          │  │ - Synapse Link     │  │
        │  └────────────────────┘  │          │  └────────────────────┘  │
        │  ┌────────────────────┐  │          │                          │
        │  │ Notebooks          │  │          │                          │
        │  │ - ELT workflows    │  │          │                          │
        │  │ - Data validation  │  │          │                          │
        │  │ - Monitoring       │  │          │                          │
        │  └────────────────────┘  │          │                          │
        │  ┌────────────────────┐  │          │                          │
        │  │ Jobs & Workflows   │  │          │                          │
        │  │ - Scheduled runs   │  │          │                          │
        │  │ - Error handling   │  │          │                          │
        │  └────────────────────┘  │          │                          │
        └──────────────┬───────────┘          └──────────────┬───────────┘
                       │                                     │
                       └──────────────┬──────────────────────┘
                                      │
                ┌─────────────────────┴──────────────────────┐
                │                                            │
                ▼                                            ▼
    ┌──────────────────────────────┐      ┌──────────────────────────────┐
    │   BI TOOLS & DASHBOARDS      │      │   MACHINE LEARNING          │
    │  ┌────────────────────────┐  │      │  ┌────────────────────────┐  │
    │  │ Power BI               │  │      │  │ MLflow                 │  │
    │  │ - Star Schema Reports  │  │      │  │ - Model Tracking       │  │
    │  │ - Real-time Dashboards │  │      │  │ - Experiment Tracking  │  │
    │  └────────────────────────┘  │      │  └────────────────────────┘  │
    │  ┌────────────────────────┐  │      │  ┌────────────────────────┐  │
    │  │ Tableau / Looker       │  │      │  │ AutoML / Spark ML      │  │
    │  │ - Custom Visualizations│  │      │  │ - Customer Segmentation│  │
    │  │ - Ad-hoc Analysis      │  │      │  │ - Churn Prediction     │  │
    │  └────────────────────────┘  │      │  │ - Sales Forecasting    │  │
    │  ┌────────────────────────┐  │      │  └────────────────────────┘  │
    │  │ Excel / Python         │  │      │                              │
    │  │ - Connected Analytics  │  │      │                              │
    │  └────────────────────────┘  │      │                              │
    └──────────────────────────────┘      └──────────────────────────────┘
                                          
    ┌──────────────────────────────────────────────────────────────────┐
    │              MONITORING, SECURITY & GOVERNANCE                   │
    │  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌─────────────┐  │
    │  │   Azure    │ │  Databricks│ │   Azure    │ │    Azure    │  │
    │  │ Monitor +  │ │  Runtime   │ │   Purview  │ │   Key Vault │  │
    │  │   Alerts   │ │  Logging   │ │  (Catalog) │ │  (Secrets)  │  │
    │  └────────────┘ └────────────┘ └────────────┘ └─────────────┘  │
    └──────────────────────────────────────────────────────────────────┘
```

---

## 1. Components & Services

### 1.1 Data Ingestion Layer

#### Azure Data Factory (ADF)
**Purpose:** Orchestrate and automate data pipelines from source systems to cloud

| Component | Description | Use Case |
|-----------|-------------|----------|
| Copy Activity | Bulk data transfer | Initial load of CSV files |
| Mapping Data Flow | Visual ETL transformations | Data cleansing & validation |
| Pipeline Triggers | Scheduled/event-based execution | Daily/weekly data imports |
| Linked Services | Connection management | Secure connections to sources |

**Pipeline Design:**
```
CSV Source (Blob Storage) 
  → Validation Activity 
  → Mapping Data Flow (Transform) 
  → ADLS2 (Bronze Layer) 
  → Notification (Success/Failure)
```

#### Azure Blob Storage
- **Purpose:** Staging area for incoming CSV files
- **Configuration:** 
  - Container: `raw-data-source`
  - Retention: 30 days (after processing)
  - Access: Private with SAS tokens

### 1.2 Storage Layer (Lakehouse Pattern)

#### Azure Data Lake Storage Gen2 (ADLS2)
**Three-Layer Architecture (Bronze-Silver-Gold):**

```
/retail-sales-lakehouse/
├── bronze/
│   ├── customer/
│   │   └── customer_YYYYMMDD_hhmmss.csv
│   ├── product/
│   │   └── product_YYYYMMDD_hhmmss.csv
│   ├── sales/
│   │   └── sales_returns_YYYYMMDD_hhmmss.csv
│   └── payment/
│       └── payment_refund_YYYYMMDD_hhmmss.csv
│
├── silver/
│   ├── customer/
│   │   └── customer_clean/ (Parquet/Delta)
│   ├── product/
│   │   └── product_clean/ (Parquet/Delta)
│   ├── sales/
│   │   └── sales_clean/ (Parquet/Delta)
│   └── payment/
│       └── payment_clean/ (Parquet/Delta)
│
└── gold/
    ├── analytics/
    │   ├── dim_customer/ (Delta Table)
    │   ├── dim_product/ (Delta Table)
    │   ├── fact_sales/ (Delta Table)
    │   ├── fact_payment/ (Delta Table)
    │   └── agg_daily_sales/ (Delta Table)
    └── reports/
        ├── sales_summary/ (Parquet)
        └── customer_metrics/ (Parquet)
```

**Access Control:**
- Bronze: Data Engineers
- Silver: Data Analysts & Engineers
- Gold: BI/Analytics & End Users

### 1.3 Processing Layer - Databricks

#### Cluster Configuration
```yaml
Cluster Name: "retail-sales-prod"
Runtime: Databricks Runtime 13.3 LTS (Spark 3.4.0)
Node Type: i3.xlarge (4 CPU, 30.5 GB RAM)
Driver: 1x i3.xlarge
Workers: 2-8 (auto-scaling)
Max Workers: 8
Storage: 200 GB (SSD)
```

#### Delta Lake Tables
Delta Lake provides ACID transactions and time-travel capabilities.

**Bronze → Silver Transformations (PySpark):**
```python
# Customer dimension
from delta.tables import DeltaTable
from pyspark.sql.functions import *

# Read raw data
raw_customer = spark.read.csv(
    "abfss://bronze@datalake.dfs.core.windows.net/customer/",
    header=True, inferSchema=True
)

# Clean & standardize
clean_customer = raw_customer \
    .dropna(subset=['customer_id', 'email']) \
    .dropDuplicates(['email']) \
    .withColumn('created_at', current_timestamp()) \
    .withColumn('data_quality_flag', 
        when(col('age_group').isin(['18-25', '26-35', '36-45', '46-55', '56-65', '65+']), 1)
        .otherwise(0))

# Write to Silver
clean_customer.write \
    .format("delta") \
    .mode("overwrite") \
    .option("path", "abfss://silver@datalake.dfs.core.windows.net/customer/customer_clean") \
    .save()
```

**Silver → Gold (Star Schema):**
```python
# Fact table: SALES
from pyspark.sql.window import Window

# Read from Silver
customer_silver = spark.read.delta("abfss://silver@datalake.dfs.core.windows.net/customer/customer_clean")
product_silver = spark.read.delta("abfss://silver@datalake.dfs.core.windows.net/product/product_clean")
sales_silver = spark.read.delta("abfss://silver@datalake.dfs.core.windows.net/sales/sales_clean")

# Create fact table with surrogate keys
fact_sales = sales_silver \
    .join(customer_silver.select('customer_id'), 'customer_id', 'left') \
    .join(product_silver.select('product_id'), 'product_id', 'left') \
    .withColumn('sales_id', row_number().over(Window.orderBy('order_date'))) \
    .withColumn('sales_total', 
        (col('quantity') * col('unit_price')) - col('discount_amount') + col('tax_amount')) \
    .select('sales_id', 'customer_id', 'product_id', 'order_date', 
            'quantity', 'unit_price', 'discount_amount', 'tax_amount', 
            'sales_total', 'sales_channel', 'order_status', 'is_returned',
            'return_date', 'return_reason', 'return_quantity', 'return_amount')

# Write to Gold (Delta)
fact_sales.write \
    .format("delta") \
    .mode("overwrite") \
    .partitionBy("order_date") \
    .option("path", "abfss://gold@datalake.dfs.core.windows.net/analytics/fact_sales") \
    .save()
```

#### Databricks Workflows (Jobs)
```yaml
Job 1: "Daily_Data_Ingestion"
  Trigger: Daily @ 02:00 AM UTC
  Tasks:
    - Task 1: Run ADF Pipeline (via REST API)
    - Task 2: Validate Bronze Layer Data
    - Task 3: Notify on completion
  Timeout: 30 minutes
  Retries: 2

Job 2: "Bronze_to_Silver_Transform"
  Trigger: After Job 1 completion
  Tasks:
    - Task 1: Run Notebook: "02_bronze_to_silver_customer"
    - Task 2: Run Notebook: "02_bronze_to_silver_product"
    - Task 3: Run Notebook: "02_bronze_to_silver_sales"
    - Task 4: Run Notebook: "02_bronze_to_silver_payment"
    - Task 5: Data Quality Checks
  Timeout: 45 minutes

Job 3: "Silver_to_Gold_Analytics"
  Trigger: After Job 2 completion
  Tasks:
    - Task 1: Run Notebook: "03_silver_to_gold_dimensions"
    - Task 2: Run Notebook: "03_silver_to_gold_facts"
    - Task 3: Generate Aggregates
    - Task 4: Refresh Power BI Models
  Timeout: 60 minutes
```

#### Databricks SQL Warehouse
```sql
-- Create Gold Layer Tables (via Databricks SQL)

CREATE TABLE IF NOT EXISTS gold.dim_customer
USING DELTA
LOCATION 'abfss://gold@datalake.dfs.core.windows.net/analytics/dim_customer'
AS
SELECT * FROM silver.customer_clean;

CREATE TABLE IF NOT EXISTS gold.dim_product
USING DELTA
LOCATION 'abfss://gold@datalake.dfs.core.windows.net/analytics/dim_product'
AS
SELECT * FROM silver.product_clean;

CREATE TABLE IF NOT EXISTS gold.fact_sales
USING DELTA
PARTITIONED BY (order_date)
LOCATION 'abfss://gold@datalake.dfs.core.windows.net/analytics/fact_sales'
AS
SELECT 
    row_number() OVER (ORDER BY order_date) as sales_id,
    customer_id, product_id, order_date, delivery_date,
    quantity, unit_price, discount_amount, tax_amount,
    sales_total, sales_channel, order_status,
    is_returned, return_date, return_reason, return_quantity, return_amount,
    current_timestamp() as load_timestamp
FROM silver.sales_clean;

CREATE TABLE IF NOT EXISTS gold.fact_payment
USING DELTA
PARTITIONED BY (transaction_date)
LOCATION 'abfss://gold@datalake.dfs.core.windows.net/analytics/fact_payment'
AS
SELECT 
    row_number() OVER (ORDER BY transaction_date) as payment_id,
    sales_id, payment_type, card_brand, amount,
    payment_status, transaction_date,
    is_refunded, refund_amount, refund_reason, refund_status, refund_date,
    current_timestamp() as load_timestamp
FROM silver.payment_clean;

-- Aggregate tables for fast BI queries
CREATE TABLE IF NOT EXISTS gold.agg_daily_sales
USING DELTA
PARTITIONED BY (sales_date)
AS
SELECT 
    DATE(order_date) as sales_date,
    sales_channel,
    SUM(sales_total) as total_revenue,
    SUM(quantity) as total_quantity,
    COUNT(DISTINCT sales_id) as total_orders,
    COUNT(DISTINCT customer_id) as unique_customers,
    SUM(CASE WHEN is_returned = 1 THEN 1 ELSE 0 END) as return_count,
    AVG(sales_total) as avg_order_value
FROM gold.fact_sales
GROUP BY DATE(order_date), sales_channel;
```

### 1.4 Data Warehouse Layer

#### Azure SQL Database (Data Warehouse)
**Configuration:**
```
Service Tier: Premium (P6)
Compute: 1000 DTUs
Storage: 1 TB
Backup: Geo-redundant (RA-GRS)
```

**Synchronized Tables:**
- CUSTOMER (Dimension)
- PRODUCT (Dimension)
- SALES (Fact) - partitioned by month
- PAYMENT (Fact) - partitioned by month

**Sync Method: Databricks → Azure SQL**
```python
# Write Databricks Delta table to Azure SQL Database
df.write \
    .format("com.microsoft.sqlserver.jdbc.spark") \
    .mode("overwrite") \
    .option("url", "jdbc:sqlserver://server.database.windows.net:1433;database=salesdb") \
    .option("dbtable", "dbo.fact_sales") \
    .option("user", "admin@server") \
    .option("password", dbutils.secrets.get(scope="azure-keyvault", key="sql-password")) \
    .save()
```

### 1.5 Analytics & BI Layer

#### Power BI
**Connections:**
- Direct connection to Azure SQL Database (Star Schema)
- Direct connection to Databricks SQL Warehouse

**Data Model:**
```
Relationships:
  fact_sales → dim_customer (customer_id)
  fact_sales → dim_product (product_id)
  fact_sales ← fact_payment (sales_id)
```

**Sample Dashboards:**
1. **Sales Performance Dashboard**
   - Revenue by Channel, Product, Customer Segment
   - KPIs: Total Sales, Average Order Value, Customer Count
   - Trends: Month-over-Month, Year-over-Year

2. **Product Analysis Dashboard**
   - Top/Bottom Products by Revenue
   - Return Rate by Product Category
   - Inventory & Reorder Analysis

3. **Customer Analytics Dashboard**
   - Customer Lifetime Value
   - Segmentation by Spending Tier
   - Churn Prediction & Retention Metrics

4. **Payment & Returns Dashboard**
   - Payment Success Rate by Type & Brand
   - Refund Analysis & Trends
   - Payment Fraud Detection

---

## 2. Data Ingestion Pipelines

### 2.1 ADF Pipeline: Load Customer Data
```json
{
  "name": "Pipeline_Load_Customer",
  "properties": {
    "activities": [
      {
        "name": "Copy_Customer_CSV",
        "type": "Copy",
        "inputs": [
          {
            "referenceName": "AzureBlobStorage_CSV",
            "parameters": { "filename": "customer_*.csv" }
          }
        ],
        "outputs": [
          {
            "referenceName": "ADLS2_Bronze_Customer"
          }
        ],
        "typeProperties": {
          "source": { "type": "DelimitedTextSource" },
          "sink": { "type": "ParquetSink" },
          "translator": {
            "mappings": [
              { "source": "customer_id", "sink": "customer_id" },
              { "source": "name", "sink": "name" },
              { "source": "email", "sink": "email" }
            ]
          }
        }
      },
      {
        "name": "Validation_Activity",
        "type": "ExecuteDataFlow",
        "dependsOn": [
          { "activity": "Copy_Customer_CSV", "dependencyConditions": ["Succeeded"] }
        ]
      }
    ]
  }
}
```

### 2.2 Databricks Notebook: Bronze → Silver

**Notebook Location:** `/Workspace/etl/02_bronze_to_silver_customer`

```python
# Databricks notebook source
# COMMAND
# Bronze to Silver: Customer Data Cleaning

from pyspark.sql.functions import *
from pyspark.sql.types import *
import logging

# Initialize logger
logger = logging.getLogger(__name__)

# COMMAND
# Read Bronze data
bronze_path = "abfss://bronze@datalake.dfs.core.windows.net/customer/"
df_bronze = spark.read.parquet(bronze_path)

logger.info(f"Bronze records: {df_bronze.count()}")

# COMMAND
# Data Quality Checks
df_bronze.createOrReplaceTempView("bronze_customer")

# Check for duplicates
duplicates = spark.sql("""
  SELECT email, COUNT(*) as cnt 
  FROM bronze_customer 
  GROUP BY email 
  HAVING cnt > 1
""")

logger.warning(f"Duplicate emails found: {duplicates.count()}")

# COMMAND
# Transformation: Cleaning & Standardization
df_silver = df_bronze \
    .dropna(subset=['customer_id', 'email']) \
    .dropDuplicates(['email']) \
    .withColumn('email', lower(trim(col('email')))) \
    .withColumn('name', initcap(trim(col('name')))) \
    .withColumn('phone', regexp_replace(col('phone'), '[^0-9+]', '')) \
    .withColumn('postal_code', regexp_replace(col('postal_code'), '[^0-9A-Z-]', '')) \
    .withColumn('created_at', to_timestamp(col('created_at'), 'yyyy-MM-dd HH:mm:ss')) \
    .withColumn('load_timestamp', current_timestamp()) \
    .withColumn('data_quality_score',
        when(col('age_group').isin(['18-25', '26-35', '36-45', '46-55', '56-65', '65+']), 1)
        .when(col('gender').isin(['M', 'F', 'Other']), 1)
        .otherwise(0)
    )

# COMMAND
# Write to Silver (Delta format)
silver_path = "abfss://silver@datalake.dfs.core.windows.net/customer/customer_clean"

df_silver.write \
    .format("delta") \
    .mode("overwrite") \
    .option("path", silver_path) \
    .save()

logger.info(f"Silver records written: {df_silver.count()}")

# COMMAND
# Audit log
spark.sql(f"""
  INSERT INTO audit_log (pipeline, layer, table_name, record_count, status, timestamp)
  VALUES ('ETL', 'Silver', 'customer_clean', {df_silver.count()}, 'SUCCESS', current_timestamp())
""")
```

---

## 3. Data Quality & Validation

### 3.1 Data Quality Rules (Databricks)
```python
# Data Quality Framework using Delta Lake expectations
from delta.tables import DeltaTable

def validate_customer_data(df, log_path):
    """Validate customer dimension data"""
    
    validations = {
        "null_customer_ids": df.filter(col("customer_id").isNull()).count() == 0,
        "unique_emails": df.count() == df.dropDuplicates(["email"]).count(),
        "valid_age_groups": df.filter(
            col("age_group").isin(['18-25', '26-35', '36-45', '46-55', '56-65', '65+'])
        ).count() == df.count(),
        "valid_genders": df.filter(
            col("gender").isin(['M', 'F', 'Other', 'Prefer not to say'])
        ).count() == df.count()
    }
    
    failed_checks = [k for k, v in validations.items() if not v]
    
    if failed_checks:
        raise ValueError(f"Data quality checks failed: {failed_checks}")
    
    return True
```

### 3.2 Monitoring & Alerting
```python
# Monitor fact table partitions
from databricks.sql import sql

def check_partition_freshness(table_name, hours_threshold=24):
    """Check if latest partition is within threshold"""
    
    query = f"""
    SELECT 
        MAX(order_date) as latest_partition,
        CURRENT_DATE - MAX(order_date) as days_behind
    FROM {table_name}
    """
    
    result = spark.sql(query).collect()[0]
    days_behind = result["days_behind"]
    
    if days_behind > (hours_threshold / 24):
        # Send alert to Azure Monitor
        alert_metric = f"Table {table_name} is {days_behind} days behind"
        print(f"ALERT: {alert_metric}")
        return False
    
    return True
```

---

## 4. Security & Governance

### 4.1 Azure Key Vault Integration
```python
# Store sensitive credentials in Azure Key Vault

# Database connection
sql_password = dbutils.secrets.get(scope="azure-keyvault", key="sql-db-password")
storage_key = dbutils.secrets.get(scope="azure-keyvault", key="adls-storage-key")

# Use credentials
connection_string = f"Driver={{ODBC Driver 17 for SQL Server}};Server=tcp:server.database.windows.net,1433;Database=salesdb;Uid=admin;Pwd={sql_password};"
```

### 4.2 Access Control (RBAC)
```
Azure RBAC:
├── Data Owner
│   ├── All ADLS2 access (Read/Write/Delete)
│   ├── Databricks workspace admin
│   └── Azure SQL Database admin
├── Data Engineer
│   ├── Bronze/Silver read-write access
│   ├── Databricks cluster creation
│   └── Pipeline execution
├── Data Analyst
│   ├── Silver/Gold read-only access
│   ├── Databricks SQL warehouse access
│   └── Power BI connection
└── Business User
    ├── Gold read-only access
    └── Power BI dashboards only
```

### 4.3 Data Encryption
- **At Rest:** Azure Storage encryption (AES-256)
- **In Transit:** TLS 1.2+ for all communications
- **Database:** Azure SQL Transparent Data Encryption (TDE)

### 4.4 Audit & Compliance
```python
# Log all data access and transformations
spark.sql("""
  CREATE TABLE IF NOT EXISTS audit_log (
    audit_id BIGINT GENERATED ALWAYS AS IDENTITY,
    user_id STRING,
    action STRING,
    table_name STRING,
    timestamp TIMESTAMP,
    row_count LONG,
    status STRING,
    error_message STRING
  )
""")
```

---

## 5. Deployment Architecture

### 5.1 Development → Production Flow

```
┌──────────────┐
│ Development  │ (Developer notebooks, test data)
├──────────────┤
│   Git/ADO    │ (Version control)
└────────┬─────┘
         │
         ▼
┌──────────────┐
│  Staging     │ (Full testing, realistic data volume)
├──────────────┤
│  ADLS2 (SL)  │
└────────┬─────┘
         │
         ▼
┌──────────────┐
│  Production  │ (Live data, HA/DR)
├──────────────┤
│  ADLS2 (PR)  │
└──────────────┘
```

### 5.2 Infrastructure as Code (Terraform)

**Provider: Azure + Databricks**

```hcl
# main.tf - Core Infrastructure

terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
    databricks = {
      source  = "databricks/databricks"
      version = "~> 1.0"
    }
  }
}

provider "azurerm" {
  features {}
}

provider "databricks" {
  host  = azurerm_databricks_workspace.this.workspace_url
  token = databricks_token.pat.token_value
}

# Resource Group
resource "azurerm_resource_group" "retail_analytics" {
  name     = "rg-retail-analytics-prod"
  location = "East US 2"
}

# Storage Account (Data Lake)
resource "azurerm_storage_account" "datalake" {
  name                     = "datalakeretailsales"
  resource_group_name      = azurerm_resource_group.retail_analytics.name
  location                 = azurerm_resource_group.retail_analytics.location
  account_tier             = "Standard"
  account_replication_type = "GRS"
  is_hns_enabled          = true  # Enable Data Lake Gen2
}

# Containers (Bronze, Silver, Gold)
resource "azurerm_storage_container" "bronze" {
  name                  = "bronze"
  storage_account_name  = azurerm_storage_account.datalake.name
  container_access_type = "private"
}

resource "azurerm_storage_container" "silver" {
  name                  = "silver"
  storage_account_name  = azurerm_storage_account.datalake.name
  container_access_type = "private"
}

resource "azurerm_storage_container" "gold" {
  name                  = "gold"
  storage_account_name  = azurerm_storage_account.datalake.name
  container_access_type = "private"
}

# Data Factory
resource "azurerm_data_factory" "adf" {
  name                = "adf-retail-sales-prod"
  location            = azurerm_resource_group.retail_analytics.location
  resource_group_name = azurerm_resource_group.retail_analytics.name
}

# Azure SQL Database
resource "azurerm_mssql_server" "sqlserver" {
  name                         = "sqlsrv-retail-sales-prod"
  resource_group_name          = azurerm_resource_group.retail_analytics.name
  location                     = azurerm_resource_group.retail_analytics.location
  version                      = "12.0"
  administrator_login          = "sqladmin"
  administrator_login_password = random_password.sql_password.result
}

resource "azurerm_mssql_database" "salesdb" {
  name           = "salesdb"
  server_id      = azurerm_mssql_server.sqlserver.id
  collation      = "SQL_Latin1_General_CP1_CI_AS"
  sku_name       = "P6"
  zone_redundant = true
}

# Databricks Workspace
resource "azurerm_databricks_workspace" "databricks" {
  name                = "dbw-retail-sales-prod"
  resource_group_name = azurerm_resource_group.retail_analytics.name
  location            = azurerm_resource_group.retail_analytics.location
  sku                 = "Premium"
}

# Databricks Cluster
resource "databricks_cluster" "retail_prod_cluster" {
  cluster_name            = "retail-sales-prod"
  spark_version           = "13.3.x-scala2.12"
  node_type_id            = "i3.xlarge"
  driver_node_type_id     = "i3.xlarge"
  num_workers             = 2
  autotermination_minutes = 20
  
  azure_attributes {
    availability           = "SPOT"
    first_on_demand        = 1
    spot_bid_max_price     = -1  # Use spot pricing
  }
}

output "databricks_host" {
  value = azurerm_databricks_workspace.databricks.workspace_url
}

output "storage_account_url" {
  value = azurerm_storage_account.datalake.primary_blob_endpoint
}
```

---

## 6. Implementation Roadmap

### Phase 1: Foundation (Week 1-2)
- [ ] Set up Azure Resource Group & subscriptions
- [ ] Create ADLS2 with Bronze/Silver/Gold containers
- [ ] Configure Databricks workspace & cluster
- [ ] Set up Azure Key Vault for secrets
- [ ] Establish git repository & CI/CD pipeline

### Phase 2: Data Ingestion (Week 3-4)
- [ ] Create ADF pipelines for CSV ingestion
- [ ] Build Databricks notebooks for Bronze layer
- [ ] Implement data validation rules
- [ ] Set up monitoring & alerting

### Phase 3: Transformation (Week 5-6)
- [ ] Develop Bronze → Silver transformation notebooks
- [ ] Create Delta Lake tables in Silver layer
- [ ] Build data quality checks & monitoring
- [ ] Test incremental loading logic

### Phase 4: Analytics Layer (Week 7-8)
- [ ] Create Star Schema tables (Gold layer)
- [ ] Build aggregation tables for BI
- [ ] Sync Gold tables to Azure SQL Database
- [ ] Set up Databricks SQL Warehouse for queries

### Phase 5: BI & Reporting (Week 9-10)
- [ ] Connect Power BI to Azure SQL & Databricks
- [ ] Build sample dashboards (Sales, Products, Customer, Payment)
- [ ] Implement row-level security (RLS)
- [ ] Deploy Power BI Premium workspace

### Phase 6: ML & Advanced Analytics (Week 11-12)
- [ ] Set up MLflow for model tracking
- [ ] Build customer segmentation model
- [ ] Create churn prediction model
- [ ] Implement sales forecasting

### Phase 7: Production Hardening (Week 13-14)
- [ ] Load testing & performance tuning
- [ ] Security & compliance audits
- [ ] Disaster recovery & backup testing
- [ ] Documentation & knowledge transfer

---

## 7. Cost Optimization

### 7.1 Azure Pricing Estimates (Monthly)

| Service | Configuration | Cost |
|---------|---------------|------|
| ADLS2 | 1 TB storage | $20 |
| Azure Data Factory | 10 activities/day | $150 |
| Azure SQL Database | P6 (1000 DTUs) | $2,400 |
| Databricks | 8 cores/month (on-demand) | $800 |
| Power BI Premium | 1 capacity (P1) | $4,000 |
| **Total** | | **~$7,370/month** |

### 7.2 Cost Reduction Strategies
- Use **Spot VMs** for Databricks clusters (50-70% savings)
- Archive cold data to Blob Storage (Standard tier)
- Use **Databricks SQL Warehouses** instead of All-Purpose clusters
- Implement **auto-termination** for idle clusters
- Schedule non-production environments to run only during business hours

---

## 8. Monitoring & Operations

### 8.1 Health Check Dashboard (Azure Monitor)

```
Metrics to Monitor:
├── Data Ingestion
│   ├── CSV file arrival rate
│   ├── ADF pipeline success rate
│   └── Data volume (GB/day)
├── Processing
│   ├── Databricks job completion time
│   ├── Spark cluster health
│   └── Delta table size
├── Quality
│   ├── Null values %
│   ├── Duplicate record count
│   └── Data freshness (hours behind)
└── Warehouse
    ├── Query response time
    ├── Database CPU/DTU usage
    └── Connection pool usage
```

### 8.2 Alerting Rules
```yaml
Alert: "High ADF Pipeline Failure Rate"
  Condition: Failure rate > 10% in last 24 hours
  Action: Email + Slack notification
  Severity: Critical

Alert: "Data Freshness SLA Breach"
  Condition: Latest partition > 30 hours old
  Action: Page on-call engineer
  Severity: High

Alert: "SQL Database DTU Over 80%"
  Condition: DTU usage > 80% for 5 minutes
  Action: Scale up database or kill long queries
  Severity: Medium
```

---

## 9. Sample Queries (Databricks SQL)

### 9.1 Sales Performance
```sql
SELECT 
    DATE_TRUNC('day', order_date) as sales_date,
    sales_channel,
    COUNT(DISTINCT sales_id) as num_orders,
    SUM(sales_total) as total_revenue,
    AVG(sales_total) as avg_order_value,
    SUM(CASE WHEN is_returned = 1 THEN 1 ELSE 0 END) as returns,
    ROUND(SUM(CASE WHEN is_returned = 1 THEN 1 ELSE 0 END) / COUNT(*) * 100, 2) as return_rate
FROM gold.fact_sales
WHERE order_date >= DATE_SUB(CURRENT_DATE, 30)
GROUP BY DATE_TRUNC('day', order_date), sales_channel
ORDER BY sales_date DESC, total_revenue DESC;
```

### 9.2 Customer Lifetime Value
```sql
SELECT 
    c.customer_id,
    c.name,
    c.customer_segment,
    COUNT(DISTINCT s.sales_id) as total_orders,
    SUM(s.sales_total) as lifetime_value,
    MAX(s.order_date) as last_purchase_date,
    DATEDIFF(CURRENT_DATE, MAX(s.order_date)) as days_since_purchase,
    AVG(s.sales_total) as avg_order_value
FROM gold.dim_customer c
LEFT JOIN gold.fact_sales s ON c.customer_id = s.customer_id
GROUP BY c.customer_id, c.name, c.customer_segment
HAVING COUNT(DISTINCT s.sales_id) > 0
ORDER BY lifetime_value DESC;
```

### 9.3 Product Return Analysis
```sql
SELECT 
    p.product_id,
    p.name,
    p.category,
    COUNT(DISTINCT s.sales_id) as total_sales,
    SUM(s.quantity) as total_units_sold,
    SUM(CASE WHEN s.is_returned = 1 THEN s.return_quantity ELSE 0 END) as units_returned,
    ROUND(SUM(CASE WHEN s.is_returned = 1 THEN s.return_quantity ELSE 0 END) / 
          SUM(s.quantity) * 100, 2) as return_rate_pct,
    SUM(s.sales_total) as total_revenue,
    SUM(CASE WHEN s.is_returned = 1 THEN s.return_amount ELSE 0 END) as total_refunds,
    ROUND(SUM(CASE WHEN s.is_returned = 1 THEN s.return_amount ELSE 0 END) / 
          SUM(s.sales_total) * 100, 2) as refund_rate_pct
FROM gold.dim_product p
LEFT JOIN gold.fact_sales s ON p.product_id = s.product_id
GROUP BY p.product_id, p.name, p.category
ORDER BY return_rate_pct DESC, total_sales DESC;
```

---

## 10. Troubleshooting Guide

### Common Issues & Solutions

| Issue | Cause | Solution |
|-------|-------|----------|
| ADF pipeline timeout | Large file or network issue | Increase timeout, split into smaller batches |
| Databricks cluster won't start | Quota exceeded or subnet issue | Check Azure quotas, verify VNet settings |
| Delta table write failures | Schema mismatch | Use `mergeSchema=true` option |
| Slow Power BI queries | Large fact table without aggregates | Create aggregate/summary tables, use materialized views |
| Out of memory errors | Shuffles with large joins | Broadcast smaller dimension tables, increase executor memory |

---

## 11. Conclusion

This architecture provides:
- **Scalability:** Auto-scaling clusters handle 10x data growth
- **Cost-efficiency:** Spot VMs + reserved capacity = 40% savings
- **Reliability:** 99.95% SLA with geo-redundancy
- **Flexibility:** Support for batch & real-time analytics
- **Security:** Enterprise-grade encryption, RBAC, audit logs

**Next Steps:**
1. Provision Azure resources using Terraform
2. Deploy ADF pipelines for data ingestion
3. Build & test Databricks notebooks
4. Create Power BI dashboards
5. Implement monitoring & alerting
