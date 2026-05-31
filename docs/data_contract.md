# Company Risk Intelligence Platform – Data Contract

This document defines the data contract for the Company Risk Intelligence Platform. It specifies ownership, source systems, schemas, quality expectations, security requirements, lineage, orchestration, and consumption rules for all datasets produced within the platform.

---

# Table of Contents

* [Data Contract](#data-contract)
* [Data Ownership](#data-ownership)
* [Source System Details](#source-system-details)
* [Source Table Details](#source-table-details)

  * [company_registry](#company_registry)
  * [stock_market_data](#stock_market_data)
  * [financial_news](#financial_news)
* [Business Definitions](#business-definitions)
* [Target (Gold Layer Design)](#target-gold-layer-design)
* [Data Access & Security](#data-access--security)
* [Confidentiality Classification](#confidentiality-classification)
* [Data Refresh & Latency](#data-refresh--latency)
* [Data Quality Rules](#data-quality-rules)
* [Data Dictionary](#data-dictionary)

  * [dim_company](#dim_company)
  * [dim_date](#dim_date)
  * [dim_news_event](#dim_news_event)
  * [fact_stock_performance](#fact_stock_performance)
  * [fact_company_risk](#fact_company_risk)
* [Lineage](#lineage)
* [Orchestration Details](#orchestration-details)
* [Consumption Layer](#consumption-layer)

---

# Data Contract

```yaml
data_product: company_risk_intelligence_platform

domain: financial_risk_analytics

description: >
  This data product provides integrated company intelligence datasets
  by combining company registry information, stock market performance,
  and financial news signals to support company risk assessment,
  governance monitoring, and business intelligence analytics.
```

---

# Data Ownership

```yaml
data_owner:
  business_owner: Financial Risk Analytics Team
  technical_owner: Data Engineering Team
  steward: Data Governance Team
```

### Responsibilities

| Role            | Responsibility                                                        |
| --------------- | --------------------------------------------------------------------- |
| Business Owner  | Defines business KPIs, risk metrics, and reporting requirements       |
| Technical Owner | Builds, maintains, and monitors data pipelines                        |
| Data Steward    | Ensures governance, data quality, compliance, and metadata management |

---

# Source System Details

```yaml
source_systems:

  - name: Companies House API
    type: REST API
    ingestion_method: Batch

  - name: Yahoo Finance API
    type: REST API
    ingestion_method: Batch

  - name: Yahoo Finance News
    type: Financial News Feed
    ingestion_method: Batch
```

### Source Overview

| Source             | Purpose                                         |
| ------------------ | ----------------------------------------------- |
| Companies House    | Company registration and governance information |
| Yahoo Finance      | Stock prices, volume, and market metrics        |
| Yahoo Finance News | Company-related news and events                 |

---

# Source Table Details

## company_registry

```yaml
company_registry:

  grain: one row per registered company

  primary_key: company_number

  columns:
    - company_number
    - company_name
    - company_status
    - incorporation_date
    - company_type
    - sic_code
    - registered_address
```

### Purpose

Stores official company registry information and governance-related attributes.

---

## stock_market_data

```yaml
stock_market_data:

  grain: one row per company per trading day

  primary_key:
    - ticker_symbol
    - trading_date

  columns:
    - ticker_symbol
    - trading_date
    - open_price
    - close_price
    - high_price
    - low_price
    - trading_volume
    - market_cap
```

### Purpose

Captures daily stock performance and market indicators.

---

## financial_news

```yaml
financial_news:

  grain: one row per news article

  primary_key: article_id

  columns:
    - article_id
    - company_name
    - publication_date
    - headline
    - source
    - article_url
    - sentiment_score
```

### Purpose

Captures external events and sentiment signals impacting companies.

---

# Business Definitions

```yaml
business_definitions:

  company:
    A legally registered business entity.

  stock_performance:
    Financial market activity associated with a company.

  news_event:
    External event or article that may influence company performance.

  governance_risk:
    Risk derived from registry-related attributes.

  market_risk:
    Risk derived from market behavior and stock volatility.

  sentiment_risk:
    Risk derived from negative news sentiment.

  overall_risk_score:
    Combined risk score derived from governance, market,
    and sentiment indicators.
```

---

# Target (Gold Layer Design)

```yaml
gold_layer:

  schema: gold

  tables:
    - dim_company
    - dim_date
    - dim_news_event
    - fact_stock_performance
    - fact_company_risk
```

---

## Star Schema Design

### Dimension Tables

```yaml
dim_company:
  company master data

dim_date:
  calendar attributes

dim_news_event:
  news and sentiment attributes
```

### Fact Tables

```yaml
fact_stock_performance:
  daily stock market activity

fact_company_risk:
  consolidated risk indicators and scores
```

---

# Data Access & Security

## Access Control

```yaml
access_control:

  groups:

    - ad_group_risk_analysts:
        read access

    - ad_group_data_engineers:
        full access

    - ad_group_management:
        read access

    - ad_group_governance_team:
        registry and governance access
```

---

## Security Controls

```yaml
security:

  row_level_security:
    optional

  column_level_security:
    - mask registered_address

  encryption:
    at_rest: enabled
    in_transit: TLS/HTTPS
```

---

# Confidentiality Classification

```yaml
data_classification:

  company_name: internal

  registered_address: sensitive

  stock_market_data: confidential

  risk_scores: confidential

  sentiment_data: internal
```

### Classification Rules

| Data Element       | Classification |
| ------------------ | -------------- |
| Company Name       | Internal       |
| Registered Address | Sensitive      |
| Stock Metrics      | Confidential   |
| Risk Scores        | Confidential   |
| News Sentiment     | Internal       |

---

# Data Refresh & Latency

```yaml
data_refresh:

  frequency: daily

  ingestion_time: 01:00 UTC

  availability_sla: 03:00 UTC

  latency: T+1
```

### SLA

| Metric            | Value     |
| ----------------- | --------- |
| Refresh Frequency | Daily     |
| Ingestion Time    | 01:00 UTC |
| Data Available    | 03:00 UTC |
| Processing Type   | Batch     |

---

# Data Quality Rules

```yaml
data_quality:

  rules:

    - company_number must not be null

    - company_name must not be null

    - ticker_symbol must not be null

    - trading_date must not be null

    - close_price >= 0

    - trading_volume >= 0

    - sentiment_score between -1 and 1

    - company_id must exist in dim_company

    - date_key must exist in dim_date

    - overall_risk_score between 0 and 100
```

### Quality Expectations

| Rule            | Validation |
| --------------- | ---------- |
| Company Number  | Not Null   |
| Company Name    | Not Null   |
| Trading Date    | Not Null   |
| Closing Price   | ≥ 0        |
| Trading Volume  | ≥ 0        |
| Sentiment Score | -1 to 1    |
| Risk Score      | 0 to 100   |

---

# Data Dictionary

## dim_company

```yaml
dim_company:

  company_id:
    surrogate key

  company_number:
    Companies House identifier

  company_name:
    official company name

  company_status:
    company status

  incorporation_date:
    registration date

  company_type:
    legal entity type

  sic_code:
    industry classification
```

---

## dim_date

```yaml
dim_date:

  date_key:
    YYYYMMDD

  full_date:
    calendar date

  day_num:
    day number

  month_num:
    month number

  quarter_num:
    quarter number

  year_num:
    year number
```

---

## dim_news_event

```yaml
dim_news_event:

  news_id:
    unique identifier

  headline:
    article title

  source:
    publisher

  publication_date:
    publication timestamp

  sentiment_score:
    sentiment value
```

---

## fact_stock_performance

```yaml
fact_stock_performance:

  company_id:
    FK to dim_company

  date_key:
    FK to dim_date

  ticker_symbol:
    stock ticker

  open_price:
    opening price

  close_price:
    closing price

  market_cap:
    market capitalization

  trading_volume:
    volume traded
```

---

## fact_company_risk

```yaml
fact_company_risk:

  company_id:
    FK to dim_company

  date_key:
    FK to dim_date

  governance_risk_score:
    governance indicator

  market_risk_score:
    market indicator

  sentiment_risk_score:
    sentiment indicator

  overall_risk_score:
    final calculated score
```

---

# Lineage

```yaml
lineage:

  company_registry
      -> bronze
      -> silver
      -> dim_company

  stock_market_data
      -> bronze
      -> silver
      -> fact_stock_performance

  financial_news
      -> bronze
      -> silver
      -> dim_news_event

  company_registry + stock_market_data + financial_news
      -> silver
      -> entity_resolution
      -> risk_scoring_engine
      -> fact_company_risk

  trading_date
      -> dim_date
```

---

# Orchestration Details

```yaml
orchestration:

  tool: Databricks Workflows

  pipelines:

    - ingest_company_registry

    - ingest_stock_market_data

    - ingest_financial_news

    - transform_bronze_to_silver

    - perform_entity_resolution

    - load_dim_company

    - load_dim_date

    - load_dim_news_event

    - load_fact_stock_performance

    - calculate_risk_scores

    - load_fact_company_risk
```

---

# Consumption Layer

```yaml
consumption_layer:

  users:
    - Risk Analysts
    - Financial Analysts
    - Governance Teams
    - Executive Leadership

  use_cases:
    - Company Risk Assessment
    - Financial Performance Monitoring
    - Governance Monitoring
    - News Impact Analysis
    - Cross Company Benchmarking
    - Investment Risk Evaluation

  outputs:
    - Company Intelligence Dataset
    - Risk Analytics Reports
    - Governance Reports
    - Stock Performance Reports
    - News Sentiment Reports
```

---

# End-to-End Data Flow

```text
Companies House API
            |
            v
      Bronze Layer
            |
            v
      Silver Layer
   (Cleaning & Standardization)
            |
            +-------------------+
            |                   |
            v                   v

Yahoo Finance API      Yahoo Finance News
            |                   |
            +---------+---------+
                      |
                      v

             Entity Resolution
                      |
                      v

             Risk Scoring Engine
                      |
                      v

                Gold Layer
      +-----------------------------+
      | dim_company                 |
      | dim_date                    |
      | dim_news_event              |
      | fact_stock_performance      |
      | fact_company_risk           |
      +-----------------------------+
                      |
                      v

            Analytics & Reporting
```
