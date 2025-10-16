# 🧱 Azure Data Pipeline — dbt Transformation (Medallion Architecture)

## 🚀 Project Overview

This repository contains the **dbt (Data Build Tool)** part of a full end-to-end **Azure Data Engineering project**.

The full project ingests data from **Azure SQL Database** into **Azure Data Lake Storage Gen2** using **Azure Data Factory (ADF)**, processes it with **Azure Databricks**, and transforms it into analytics-ready models using **dbt** — following the **Medallion Architecture** (Bronze → Silver → Gold).

This repo focuses on the **dbt transformations** (Silver & Gold layers).

---

## 🧩 Full Pipeline Summary

### 🔹 Step 1: Data Ingestion (Bronze Layer)

- Built using **Azure Data Factory (ADF)**.  
- The pipeline:
  1. Uses a **Lookup** activity to get a list of all tables from Azure SQL Database.  
  2. Passes those tables into a **ForEach** loop.  
  3. Inside the loop, a **Copy Data** activity moves each table from the SQL database to the **Bronze container** in **Azure Data Lake Storage Gen2**.
- Data is stored in **Parquet format** for efficiency.

---

### 🔹 Step 2: Databricks Processing (Bronze Registration)

After ingestion, a **Databricks notebook** is triggered by ADF.  
It dynamically registers each table as a Spark SQL table using parameters passed from ADF:

```python
file_name = dbutils.widgets.get('file_name')
table_schema = dbutils.widgets.get('table_schema')
table_name = dbutils.widgets.get('table_name')

spark.sql(f'CREATE DATABASE IF NOT EXISTS {table_schema}')

spark.sql(f"""
CREATE TABLE IF NOT EXISTS {table_schema}.{table_name}
USING PARQUET
LOCATION '/mnt/bronze/{file_name}/{table_schema}.{table_name}.parquet'
""")
```

✅ **Result:**  
All raw tables are available as Spark tables in the Bronze layer, ready for transformation.

---

## ⚙️ Step 3: dbt Transformations (Silver & Gold Layers)

This repository includes the **dbt project** that performs the transformation and modeling of data.

---

### 🧠 What dbt Does Here

- Cleans and standardizes raw data from the Bronze layer into **Silver**.  
- Creates **snapshots** to track historical changes in important tables.  
- Builds **fact** and **dimension** tables in the **Gold** layer for analytics.  
- Includes **data tests** to ensure data quality.

---

## 🗂️ dbt Project Structure

```text
dbt/
├── dbt_project.yml          # Main dbt configuration
├── models/
│   ├── staging/             # Clean and standardize bronze data (Silver layer)
│   ├── marts/               # Fact and dimension tables (Gold layer)
│   └── snapshots/           # Historical data tracking (SCD Type 2)
├── tests/                   # Custom data quality tests
├── macros/                  # Custom SQL macros (optional)
├── seeds/                   # Static CSV data (optional)
└── README.md
```

## ⚡ Running the dbt Project

### ⚙️ Prerequisites

Before initializing or running your dbt project, make sure you have:

- **Python 3.8+**
- **dbt-databricks** installed (the dbt adapter for Databricks)
- **Databricks CLI** installed and configured (to authenticate with Azure Databricks)

Install both using pip:

```bash
pip install dbt-databricks databricks-cli
```

Then authenticate **Databricks CLI** with your personal access token:

```bash
databricks configure --token
```

You will be prompted for:

- Databricks Host: (e.g. https://adb-xxxxxxxxx.azuredatabricks.net)

- Token: (your Databricks personal access token)

✅ Once configured, dbt can connect to your Databricks workspace using this authentication.

-------------------

### 1️⃣ Initialize dbt Project

If starting fresh, initialize a new dbt project:

```
dbt init azure_dbt_project
```

Choose databricks as the adapter during setup.

### 2️⃣ Install dbt

If not already installed:

```
pip install dbt-databricks
```

### 3️⃣ Configure your profiles.yml

Example configuration (for Databricks):

```
azure_dbt_project:
  target: dev
  outputs:
    dev:
      type: databricks
      catalog: hive_metastore
      schema: ...
      host: <your-databricks-workspace-url>
      http_path: <your-cluster-http-path>
      token: "{{ env_var('DATABRICKS_TOKEN') }}"
```

### 4️⃣ Run dbt commands
```
- dbt deps        # install packages
- dbt seed        # load any seed files (if used)
- dbt snapshot    # capture changes in tracked tables
- dbt run         # execute all models
- dbt test        # run data quality tests
- dbt docs generate && dbt docs serve  # generate and view docs
```

## 🧱 Medallion Architecture Summary

| Layer  | Description | Tool |
|--------|--------------|------|
| **Bronze** | Raw ingested data from source systems | Azure Data Factory + Azure Data Lake + Databricks |
| **Silver** | Cleaned and conformed data | dbt (staging models) |
| **Gold** | Business-ready facts and dimensions | dbt (marts models) |

---

## 🔐 Security Notes

- All credentials are stored securely in **Azure Key Vault**, **ADF Linked Services**, or **environment variables**.  
- This GitHub repository contains **only dbt code** — no secrets or access tokens are included.

---

## 🔮 Future Enhancements

- Integrate **dbt** into **ADF pipeline** for automated orchestration.  
- Add **CI/CD** with **GitHub Actions** to run dbt tests on every commit.  
- Add **data observability** (e.g., *Great Expectations* or dbt tests in production).
