# FinTech Loan Processing Data Engineering Project

## Project Overview

This project demonstrates an end-to-end Data Engineering solution built using Microsoft Azure technologies. The solution ingests loan application data from SQL Server, processes it through Bronze, Silver, and Gold layers in Azure Data Lake Storage Gen2 using Azure Data Factory, implements metadata-driven ETL, incremental loading, dimensional modeling, and Power BI reporting.

The objective of the project is to build a scalable, reusable, and production-style data platform for loan application analytics.

---

# Architecture

SQL Server
↓
Azure Data Factory
↓
Bronze Layer (Raw Parquet Files)
↓
Silver Layer (Cleansed Parquet Files)
↓
Gold Layer (Fact & Dimension Tables)
↓
Power BI Reporting

---

# Technology Stack

- Azure Data Factory (ADF)
- Azure Data Lake Storage Gen2 (ADLS Gen2)
- SQL Server
- Self Hosted Integration Runtime (SHIR)
- Power BI
- Git
- Azure DevOps Git Repository
- GitHub

---

# Source Tables

The project uses the following SQL Server source tables:

1. tbl_loan_application
2. tbl_client
3. tbl_client_address_info
4. tbl_client_bank_details
5. tbl_client_business_details

---

# Data Lake Architecture

## Bronze Layer

Purpose:
- Raw ingestion layer
- Stores source data as-is
- No transformations applied

Storage Format:
- Parquet

Bronze Tables:

- application
- client
- client_address
- client_bank
- client_business

Location:

bronze/
├── application/
├── client/
├── client_address/
├── client_bank/
└── client_business/

---

## Silver Layer

Purpose:
- Data cleansing
- Standardization
- Null handling
- Data quality improvements

Storage Format:
- Parquet

Silver Tables:

- application
- client
- client_address
- client_bank
- client_business

Location:

silver/
├── application/
├── client/
├── client_address/
├── client_bank/
└── client_business/

### Transformations Applied

Status Standardization

```sql
upper(trim(status))
```

Customer Type Standardization

```sql
upper(trim(customer_type))
```

Loan Amount Null Handling

```sql
iif(isNull(loan_amount),0.0,toDouble(loan_amount))
```

Derived Columns Created

- status_clean
- customer_type_clean
- loan_amount_clean

---

## Gold Layer

Purpose:
- Business-ready data
- Dimensional model
- Reporting layer

Storage Format:
- Parquet

Location:

gold/
├── dim_client/
├── dim_address/
├── dim_bank/
├── dim_business/
└── fact_loan_application/

---

# Data Modeling

## Fact Table

### fact_loan_application

Business Process:
- Loan Application Processing

Primary Business Key:
- id

---

## Dimension Tables

### dim_client

Source:
- tbl_client

Contains:
- Customer Information
- Personal Details
- KYC Details
- Customer Type
- Demographics

---

### dim_address

Source:
- tbl_client_address_info

Contains:
- Address Information
- City
- District
- State
- Country
- Pincode

---

### dim_bank

Source:
- tbl_client_bank_details

Contains:
- Bank Information
- Account Details
- Verification Status

---

### dim_business

Source:
- tbl_client_business_details

Contains:
- Business Details
- Industry Information
- Enterprise Information
- Income Information

---

# Relationship Mapping

Source Relationships:

tbl_loan_application.id
=
tbl_client.loan_application_id

tbl_client.id
=
tbl_client_address_info.client_id

tbl_client.id
=
tbl_client_bank_details.client_id

tbl_client.id
=
tbl_client_business_details.client_id

---

# Star Schema

                    dim_address
                         |
                         |
dim_bank ---- dim_client ---- dim_business
                         |
                         |
              fact_loan_application
<img width="1919" height="959" alt="image" src="https://github.com/user-attachments/assets/9f606eff-5693-4a97-919b-8d1818d51867" />

---

# Metadata Driven Framework

The project uses metadata-driven ingestion to avoid hardcoded pipelines.

## Metadata Table

Table:

adf_metadata

Columns:

- id
- source_table
- target_folder
- load_type
- watermark_column
- watermark_value

Example:

| id | source_table | target_folder | load_type |
|----|-------------|---------------|-----------|
| 1 | tbl_loan_application | application | Full |
| 2 | tbl_client | client | Full |
| 3 | tbl_client_address_info | client_address | Incremental |
| 4 | tbl_client_bank_details | client_bank | Incremental |
| 5 | tbl_client_business_details | client_business | Incremental |

---

# Incremental Loading Framework

Implemented using Watermark Strategy.

Columns Used:

- modified_date
- watermark_value

Metadata Columns:

- watermark_column
- watermark_value

Initial Watermark:

1900-01-01

---

# Incremental Load Logic

Step 1

Read Metadata Table

↓

Step 2

Check Load Type

If Full
→ Full Copy

If Incremental
→ Incremental Copy

↓

Step 3

Get Current Watermark

↓

Step 4

Copy Only Changed Records

```sql
WHERE modified_date > watermark_value
```

↓

Step 5

Get New Maximum Watermark

```sql
SELECT MAX(modified_date)
```

↓

Step 6

Update Metadata Table

Stored Procedure:

```sql
usp_UpdateWatermark
```

---

# Stored Procedure

```sql
CREATE PROCEDURE usp_UpdateWatermark
(
    @SourceTable VARCHAR(100),
    @Watermark DATETIME
)
AS
BEGIN

    UPDATE dbo.adf_metadata
    SET watermark_value = @Watermark
    WHERE source_table = @SourceTable

END
```

---

# Azure Data Factory Components

## Linked Services

### SQL Server

- ls_sqlserver_fintech

### Azure Data Lake Storage Gen2

- ls_adls_fintech

### Self Hosted Integration Runtime

- shir-fintech-local

---

# Datasets

## Source Datasets

- ds_sql_source
- ds_metadata_source

## Bronze Datasets

- ds_bronze_source
- ds_bronze_sink

## Silver Datasets

- ds_silver_source
- ds_silver_sink

## Gold Datasets

- ds_gold_sink

---

# Pipelines

## Bronze Pipeline

Features:

- Metadata Driven
- Dynamic Table Processing
- Full Load Support
- Incremental Load Support

Activities:

- Lookup Metadata
- ForEach
- If Condition
- Copy Data
- Watermark Update

---

## Silver Pipeline

Features:

- Metadata Driven
- Data Standardization
- Data Cleansing

Activities:

- Source
- Derived Columns
- Sink

---

## Gold Pipelines

Created Pipelines:

- pl_gold_client
- pl_gold_address
- pl_gold_bank
- pl_gold_business
- pl_gold_application

Created Data Flows:

- df_dim_client
- df_dim_address
- df_dim_bank
- df_dim_business
- df_fact_loan_application

---

# Data Flow Transformations

## Silver Layer

Derived Column:

status_clean

```sql
upper(trim(status))
```

Derived Column:

customer_type_clean

```sql
upper(trim(customer_type))
```

Derived Column:

loan_amount_clean

```sql
iif(isNull(loan_amount),0.0,toDouble(loan_amount))
```

---

# Power BI

Data Source:

- Azure Data Lake Storage Gen2
- Parquet Files

Imported Tables:

- dim_client
- dim_address
- dim_bank
- dim_business
- fact_loan_application

Implemented:

- Data Model
- Star Schema
- Relationships
- Reporting Layer

---

# Git Integration

Source Control:

- Azure DevOps Git Repository
- GitHub Repository

Git Commands Used

```bash
git pull origin main
git add .
git commit -m "Completed Bronze Silver Gold layers"
git push origin main
```

---

# Key Data Engineering Concepts Implemented

- Azure Data Factory
- Azure Data Lake Storage Gen2
- SQL Server Integration
- Self Hosted Integration Runtime
- Metadata Driven ETL
- Dynamic Pipelines
- Dynamic Datasets
- Dynamic Parameters
- Bronze Layer
- Silver Layer
- Gold Layer
- Data Cleansing
- Data Standardization
- Incremental Loading
- Watermarking
- Fact Tables
- Dimension Tables
- Star Schema
- Data Modeling
- Parquet Storage
- Power BI Reporting
- Git Version Control
- Azure DevOps Integration

---

# Future Enhancements

- Delta Lake Implementation
- Slowly Changing Dimensions (SCD Type 1 / Type 2)
- Surrogate Keys
- Partitioning Strategy
- Azure Synapse Analytics
- Azure Databricks
- CI/CD Deployment Pipelines
- Automated Monitoring & Alerting

---

# Project Outcome

Successfully built an end-to-end enterprise-style Azure Data Engineering solution implementing:

- Metadata Driven ETL
- Incremental Loading Framework
- Bronze/Silver/Gold Architecture
- Dimensional Modeling
- Star Schema Design
- Power BI Reporting
- Azure DevOps Source Control

The solution provides a scalable foundation for loan application analytics and reporting.
