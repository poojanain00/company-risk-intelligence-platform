# Business Requirements Document
## Company Risk Intelligence Platform

---

## 1. Introduction

The Company Risk Intelligence Platform is an end-to-end data engineering solution designed to integrate and analyze company-related data from multiple external sources. The platform combines company registry data, financial market data, and financial news to provide a unified view of company performance and risk.

---

## 2. Business Problem

Company data is fragmented across different systems:

- Company registry systems provide legal and structural information
- Financial platforms provide stock performance and valuation data
- News platforms provide real-time event and sentiment signals

These datasets are not natively connected, making it difficult to:

- Assess overall company health
- Identify financial or operational risks early
- Correlate market behavior with real-world events
- Perform unified company analysis

---

## 3. Project Objectives

The primary objectives of this project are:

- To integrate multiple data sources into a unified data platform
- To standardize and clean raw data into structured formats
- To resolve entity mismatches across systems (company ↔ stock ticker)
- To generate meaningful analytics and risk indicators
- To enable data-driven insights for company performance evaluation

---

## 4. Scope

### In Scope
- Integration of company registry data (Companies House)
- Integration of financial market data (Yahoo Finance)
- Integration of financial news data (Yahoo Finance News)
- Development of a structured data pipeline (Bronze, Silver, Gold layers)
- Implementation of a company risk scoring model

### Out of Scope
- Real-time streaming ingestion
- Machine learning-based prediction models
- External third-party datasets beyond defined sources

---

## 5. Stakeholders

- Data Engineers (pipeline development and maintenance)
- Data Analysts (data consumption and insights generation)
- Business Users (decision-making based on insights)

---

## 6. Data Sources

### 6.1 Companies House
Provides company registry data including:
- Company name and number
- Company status (active/dissolved)
- Incorporation date
- Industry classification (SIC codes)
- Registered address
- Officers and directors

---

### 6.2 Yahoo Finance
Provides financial market data including:
- Historical stock prices (OHLCV)
- Market capitalization
- Financial ratios (PE, EPS)
- Volatility indicators
- Trading volume

---

### 6.3 Yahoo Finance News
Provides company-related news including:
- News headlines
- Publication timestamps
- Publisher/source
- Summary of articles
- Related company ticker

---

## 7. Business Use Cases

### 7.1 Company Health Analysis
Combine legal, financial, and news data to assess overall company stability.

---

### 7.2 Risk Detection
Identify companies with:
- Negative stock trends
- High volatility
- Frequent leadership changes
- Negative news sentiment

---

### 7.3 Event Impact Analysis
Analyze how news events affect stock performance and market behavior.

---

### 7.4 Company Comparison
Compare companies based on:
- Industry
- Financial performance
- Risk level
- Market sentiment

---

## 8. Functional Requirements

### FR1: Data Ingestion
The system must ingest data from:
- Companies House API
- Yahoo Finance API (stock data)
- Yahoo Finance News

---

### FR2: Data Storage
The system must store data in three layers:
- Bronze (raw data)
- Silver (cleaned and standardized data)
- Gold (analytics-ready data)

---

### FR3: Data Transformation
The system must:
- Clean and normalize raw data
- Handle missing and duplicate records
- Standardize formats (dates, identifiers)

---

### FR4: Entity Resolution
The system must map:
- Companies House company records to Yahoo Finance ticker symbols

---

### FR5: Analytics and Insights
The system must generate:
- Stock performance metrics
- Governance indicators
- News sentiment metrics
- Company risk scores

---

## 9. Non-Functional Requirements

- Scalability: Ability to extend beyond 20 companies
- Reliability: Automated and repeatable pipeline execution
- Maintainability: Modular and well-structured codebase
- Data Quality: Validation rules and schema enforcement
- Performance: Efficient data processing for batch workloads

---

## 10. Success Criteria

The project will be successful if:

- Data is successfully ingested from all sources
- Data is cleaned and structured into Silver layer
- Entity mapping between datasets is achieved
- Gold layer provides meaningful analytics outputs
- Risk scores can be computed and interpreted

---

## 11. Deliverables

- End-to-end data pipeline
- Structured datasets (Bronze, Silver, Gold)
- Company-to-ticker mapping
- Risk scoring model
- Documentation (architecture, data contract, business requirements)

---

## 12. Future Enhancements

- Real-time data streaming
- Machine learning-based risk prediction
- Advanced sentiment analysis using NLP
- Integration with BI dashboards (Power BI, Tableau)
- Expansion to global datasets
