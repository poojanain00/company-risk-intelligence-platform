# System Architecture – Company Risk Intelligence Platform

## 1. Overview

The platform is an end-to-end data engineering system that integrates company registry data, financial market data, and financial news to generate unified company intelligence and risk insights.

The architecture follows a Medallion design pattern (Bronze → Silver → Gold) to ensure scalability, traceability, and data quality.

---

## 2. High-Level Architecture

### Data Flow
```
Companies House API  
Yahoo Finance API (Stock + News)  
        ↓  
   Ingestion Layer  
        ↓  
   Bronze Layer (Raw Data)  
        ↓  
   Silver Layer (Cleaned & Standardized Data)  
        ↓  
   Entity Resolution Layer  
        ↓  
   Gold Layer (Business Intelligence & Risk Scoring)  
        ↓  
   Analytics / Reporting Layer  
```
---

## 3. Data Layers

### 3.1 Bronze Layer (Raw Data)
- Stores unprocessed API responses
- No transformations applied
- Preserves original schema for traceability

Tables:
- bronze_companies_house
- bronze_officers
- bronze_stock_prices
- bronze_news

---

### 3.2 Silver Layer (Cleaned Data)
- Standardized schemas
- Deduplicated records
- Cleaned and validated fields
- Normalized timestamps and identifiers

Tables:
- silver_companies
- silver_officers
- silver_stock_prices
- silver_news

---

### 3.3 Entity Resolution Layer
- Maps Companies House entities to Yahoo Finance tickers
- Handles inconsistencies in naming conventions

Outputs:
- company_ticker_mapping
- enriched_company_dataset

---

### 3.4 Gold Layer (Business Intelligence)
- Aggregated and feature-engineered datasets
- Supports analytics and decision-making

Tables:
- gold_company_risk_table
- gold_stock_metrics
- gold_governance_metrics
- gold_news_impact_metrics

---

## 4. Key Design Principles

- **Modularity**: Each layer is independently maintainable
- **Scalability**: Can extend to additional data sources
- **Traceability**: Raw data preserved in Bronze layer
- **Reusability**: Clean datasets reused across analytics
- **Separation of concerns**: Ingestion, processing, and analytics are decoupled

---

## 5. Core System Components

### 5.1 Data Ingestion Pipelines
- Extract data from external APIs
- Load into Bronze layer

### 5.2 Transformation Pipelines
- Clean and standardize data
- Move data into Silver layer

### 5.3 Enrichment Pipelines
- Perform entity matching
- Link companies, stocks, and news

### 5.4 Analytics Pipelines
- Generate risk scores
- Build aggregated business metrics

---

## 6. Future Enhancements

- Real-time streaming ingestion (Kafka)
- Machine learning-based risk prediction
- Dashboard integration (Power BI / Tableau)
- Global company expansion beyond UK data
