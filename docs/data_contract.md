# Data Contract
## Company Risk Intelligence Platform

This document defines the data contract for the Company Risk Intelligence Platform: ownership, source systems, schemas, grain, keys, quality rules, lineage, and consumption. It reflects the **implemented** pipeline.

---

## Table of Contents

- [Data Product](#data-product)
- [Data Ownership](#data-ownership)
- [Source Systems](#source-systems)
- [Layered Model Overview](#layered-model-overview)
- [Bronze Layer](#bronze-layer)
- [Silver Layer](#silver-layer)
- [Gold Layer (Star Schema)](#gold-layer-star-schema)
- [Entity Resolution](#entity-resolution)
- [Business Definitions](#business-definitions)
- [Data Quality Rules](#data-quality-rules)
- [Data Classification & Security](#data-classification--security)
- [Data Refresh & Latency](#data-refresh--latency)
- [Lineage](#lineage)
- [Orchestration](#orchestration)
- [Consumption Layer](#consumption-layer)

---

## Data Product

```yaml
data_product: company_risk_intelligence_platform
domain: financial_risk_analytics
catalog: company_risk_intelligence_platform
schemas: [bronze, silver, gold]
universe: 20 UK-listed companies
description: >
  Integrated company intelligence combining registry, market, and news data
  to produce an explainable per-company risk score.
```

---

## Data Ownership

```yaml
business_owner: Financial Risk Analytics Team
technical_owner: Data Engineering Team
steward: Data Governance Team
```

| Role | Responsibility |
| --- | --- |
| Business Owner | Defines risk metrics and reporting requirements |
| Technical Owner | Builds and maintains pipelines |
| Data Steward | Governance, quality, and metadata |

---

## Source Systems

| Source | Type | Ingestion | Purpose |
| --- | --- | --- | --- |
| Companies House API | REST API | Batch | Registry & governance |
| Yahoo Finance API (yfinance) | REST API | Batch | Stock, fundamentals, company info |
| Yahoo Finance News | News feed | Batch | News & events |

---

## Layered Model Overview

```
Bronze (raw JSON → Delta)  →  Silver (cleaned, conformed)  →  Gold (star schema, scored)
```

- **Bronze** preserves raw API responses with lineage columns (`source_file`, ingestion timestamp).
- **Silver** flattens nested JSON, types columns, deduplicates on business keys, and resolves company identity.
- **Gold** aggregates Silver to one row per company and computes risk scores.

---

## Bronze Layer

Raw responses landed in Unity Catalog Volumes and read into Delta with `recursiveFileLookup`. Lineage columns (`source_file`, `ingestion_ts` / `last_update_ts`) are added; no business transformation is applied.

| Table | Source | Notes |
| --- | --- | --- |
| `bronze.ch_overview` | Companies House | nested structs for accounts, address, confirmation statement |
| `bronze.ch_people` | Companies House | `items` array of officer structs |
| `bronze.ch_filing_history` | Companies House | `items` array of filing structs; `total_count` per company |
| `bronze.yf_stock` | Yahoo Finance | daily OHLCV; ticker held in file path |
| `bronze.yf_info` | Yahoo Finance | wide company info payload |
| `bronze.yf_income_statement` | Yahoo Finance | wide: metrics in rows, periods as columns |
| `bronze.yf_balance_sheet` | Yahoo Finance | wide |
| `bronze.yf_cashflow` | Yahoo Finance | wide |
| `bronze.yf_news` | Yahoo Finance News | nested `content` struct |

---

## Silver Layer

Cleaned and conformed. Every table is joinable to a company via either `company_number` (Companies House) or a ticker (`ticker` / `ticker_safe`, Yahoo Finance), bridged by `dim_company`.

### silver.dim_company *(entity-resolution bridge)*
```yaml
grain: one row per company
primary_key: company_id        # = company_number (canonical)
columns:
  - company_id
  - company_name
  - company_number             # Companies House key
  - ticker                     # real ticker, e.g. SBRY.L  (joins yf_info)
  - ticker_safe                # path-safe ticker, e.g. SBRY_L (joins yf_stock/financials/news)
```

### silver.ch_overview
```yaml
grain: one row per company
primary_key: company_number
columns: [company_number, company_name, company_status, company_type, jurisdiction,
          date_of_creation, accounts_overdue, confirmation_overdue,
          has_been_liquidated, has_charges, has_insolvency_history, sic_codes, ...]
```

### silver.ch_people
```yaml
grain: one row per officer appointment
primary_key: [company_number, officer_name, officer_role, appointed_on]
columns: [company_number, officer_name, officer_role, appointed_on, resigned_on,
          nationality, country_of_residence, is_active, ingestion_ts]
```

### silver.ch_filing_history
```yaml
grain: one row per filing
primary_key: [company_number, transaction_id]
columns: [company_number, total_filings_lifetime, transaction_id, filing_date,
          category, filing_type, description, subcategory, paper_filed, ingestion_ts]
```

### silver.yf_info
```yaml
grain: one row per company
primary_key: ticker            # real ticker
columns: [ticker, company_name, sector, industry, country, marketcap, beta,
          overallrisk, auditrisk, boardrisk, ...]
```

### silver.yf_stock
```yaml
grain: one row per company per trading day
primary_key: [ticker, trading_date]   # NOTE: 'ticker' here holds the SAFE form
columns: [ticker, trading_date, open, high, low, close, volume, dividends,
          stock_splits, ingestion_ts]
```

### silver.yf_financials
```yaml
grain: one row per company per statement per metric per period (long format)
primary_key: [ticker_safe, statement_type, metric_name, period_end_date]
columns: [ticker_safe, statement_type, metric_name, period_end_date,
          metric_value, ingestion_ts]
statement_type: [income_statement, balance_sheet, cashflow]
```

### silver.yf_news
```yaml
grain: one row per news article
primary_key: news_id
columns: [news_id, ticker_safe, title, summary, provider_name, pub_date,
          canonical_url, ingestion_ts]
```

---

## Gold Layer (Star Schema)

A single star: one dimension and one fact at **company grain**.

```
   dim_company  1 ───< (1:1)  fact_company_risk
```

### gold.dim_company
```yaml
grain: one row per company
primary_key: company_id
columns:
  - company_id
  - company_name
  - company_number
  - ticker
  - ticker_safe
  - sector
  - industry
```

### gold.fact_company_risk
```yaml
grain: one row per company
primary_key: company_id
foreign_key: company_id -> gold.dim_company

measures:
  financial_pillar:   [debt_to_equity, current_ratio, net_margin, revenue_growth, financial_score]
  market_pillar:      [annual_volatility, max_drawdown, beta, market_score]
  governance_pillar:  [company_age_years, board_size, director_churn, accounts_overdue, governance_score]
  news_pillar:        [news_volume, news_sentiment, news_score]
  composite:          [risk_score, risk_band, score_date]
```

> **Design note:** This model extends the original three-pillar design (governance, market, sentiment) by adding a **financial** pillar, since `yf_financials` provides rich, well-populated fundamentals. Each pillar score is normalized 0–100 across the company universe; higher = higher risk.

---

## Entity Resolution

Companies House and Yahoo Finance use different identifiers. Resolution is performed in Silver via `dim_company`, which holds all identifiers for each company:

```
company_number  ──┐
ticker (real)   ──┤──>  dim_company.company_id  (canonical)
ticker_safe     ──┘
```

- Companies House tables join on `company_number`.
- `yf_info` joins on `ticker`.
- `yf_stock`, `yf_financials`, `yf_news` join on `ticker_safe`.

Mapping is a curated seed (20 companies) carrying name, number, real ticker, and safe ticker — chosen over fuzzy name matching for correctness at this scale.

---

## Business Definitions

```yaml
company:            A legally registered business entity.
financial_risk:     Risk from leverage, liquidity, profitability, and growth.
market_risk:        Risk from price volatility, drawdown, and beta.
governance_risk:    Risk from company status, age, director churn, and filing compliance.
news_risk:          Risk from news volume and sentiment.
risk_score:         Weighted 0–100 composite of the four pillars.
risk_band:          Categorical label (High / Medium / Low) derived from risk_score.
```

---

## Data Quality Rules

| Rule | Applies to |
| --- | --- |
| `company_number` not null | ch_* tables, dim_company |
| `ticker` / `ticker_safe` not null | yf_* tables |
| `trading_date` not null | yf_stock |
| `metric_value` not null and not NaN | yf_financials |
| `close >= 0`, `volume >= 0` | yf_stock |
| Deduplicate on declared business key | all Silver tables |
| `company_id` exists in `dim_company` | fact_company_risk |
| `0 <= risk_score <= 100` | fact_company_risk |
| `risk_band ∈ {High, Medium, Low}` | fact_company_risk |

---

## Data Classification & Security

| Data element | Classification |
| --- | --- |
| Company name / number | Internal |
| Registered address | Sensitive |
| Stock metrics | Confidential |
| Risk scores | Confidential |
| News sentiment | Internal |

```yaml
security:
  encryption: { at_rest: enabled, in_transit: TLS/HTTPS }
  access_control:
    data_engineers: full
    risk_analysts: read
    governance_team: registry/governance read
  secrets:
    companies_house_api_key: stored in Databricks secret scope (not in source)
```

> **Security note:** API credentials must be stored in a Databricks secret scope and read via `dbutils.secrets.get(...)`, never hardcoded in notebooks.

---

## Data Refresh & Latency

```yaml
frequency: daily (batch)
ingestion_time: 01:00 UTC
availability_sla: 03:00 UTC
latency: T+1
```

---

## Lineage

```
Companies House  -> bronze.ch_*        -> silver.ch_*        -┐
Yahoo Finance    -> bronze.yf_stock    -> silver.yf_stock    -┤
                 -> bronze.yf_info      -> silver.yf_info      ├─> dim_company (entity resolution)
                 -> bronze.yf_*_stmt    -> silver.yf_financials┤        │
Yahoo News       -> bronze.yf_news     -> silver.yf_news      -┘        │
                                                                        v
                          aggregate per pillar + normalize + weight  -> gold.fact_company_risk
                          yf_info sector/industry + dim_company       -> gold.dim_company
```

---

## Orchestration

```yaml
tool: Databricks Workflows / Notebooks
pipeline_order:
  - ingest_companies_house
  - ingest_yahoo_finance
  - ingest_yahoo_news
  - bronze_tables
  - silver_dim_company            # entity resolution
  - silver_ch_overview
  - silver_ch_people
  - silver_ch_filing_history
  - silver_yf_info
  - silver_yf_stock
  - silver_yf_financials
  - silver_yf_news
  - gold_dim_company
  - gold_fact_company_risk        # risk scoring engine
```

---

## Consumption Layer

```yaml
users: [Risk Analysts, Financial Analysts, Governance Teams, Executive Leadership]
use_cases:
  - Company risk assessment
  - Cross-company benchmarking
  - Governance monitoring
  - Financial performance review
outputs:
  - Company intelligence dataset (gold.dim_company + gold.fact_company_risk)
  - Risk ranking and pillar breakdown
```
