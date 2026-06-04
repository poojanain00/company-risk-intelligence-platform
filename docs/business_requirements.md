# Business Requirements Document
## Company Risk Intelligence Platform

> **Status:** Implemented · **Scope:** 20 UK-listed companies · **Architecture:** Medallion (Bronze → Silver → Gold) on Databricks / Delta Lake / Unity Catalog

---

## 1. Introduction

The Company Risk Intelligence Platform is an end-to-end data engineering solution that integrates company-related data from three external sources and produces a single, explainable company risk score. It combines legal registry data, financial market data, and financial news into a unified, analytics-ready data model.

The platform is implemented on Databricks using a Medallion architecture (Bronze, Silver, Gold) with data stored in Unity Catalog Volumes and Delta tables.

---

## 2. Business Problem

Company data is fragmented across systems that do not natively connect:

- **Registry systems** (Companies House) hold legal and governance information.
- **Financial platforms** (Yahoo Finance) hold stock performance and valuation data.
- **News platforms** hold event and sentiment signals.

Because these datasets use different identifiers (a Companies House number versus a stock ticker), it is difficult to assess overall company health, detect risk early, or correlate market behaviour with real-world events in one place.

---

## 3. Project Objectives

- Integrate three external data sources into one unified platform.
- Standardize and clean raw data into structured Delta tables.
- Resolve entity mismatches across systems (company number ↔ stock ticker).
- Compute a transparent, multi-factor company risk score.
- Enable data-driven company comparison and risk ranking.

---

## 4. Scope

### In Scope
- Companies House registry data (company overview, officers, filing history).
- Yahoo Finance market data (stock history, financial statements, company info).
- Yahoo Finance news data.
- A full Bronze → Silver → Gold pipeline.
- Entity resolution (company ↔ ticker mapping).
- A four-pillar risk scoring model producing a per-company composite score.

### Out of Scope
- Real-time / streaming ingestion (batch only).
- Machine-learning-based prediction.
- Data sources beyond the three defined above.
- Companies outside the defined 20-company UK universe.

---

## 5. Stakeholders

| Role | Interest |
| --- | --- |
| Data Engineers | Build and maintain the pipeline |
| Data Analysts | Consume Gold tables for insight |
| Business / Risk Users | Use risk scores for decision-making |

---

## 6. Data Sources

### 6.1 Companies House API
Registry and governance data: company name and number, company status, incorporation date, company type, SIC codes, registered address, officers/directors, and full filing history.

### 6.2 Yahoo Finance API (yfinance)
Market and fundamentals data: 1 year of daily stock history (OHLCV), annual income statement, balance sheet and cashflow, plus a company `info` payload (sector, industry, market cap, beta, valuation ratios, and Yahoo's own risk indicators).

### 6.3 Yahoo Finance News
Company-related news articles: headline, summary, publisher, publication timestamp, and URLs, nested under a `content` structure.

---

## 7. Business Use Cases

1. **Company Health Analysis** — combine legal, financial, and news signals to assess overall stability.
2. **Risk Detection** — surface companies with weak financials, high volatility, director churn, or negative news.
3. **Company Comparison** — rank and benchmark companies by pillar and composite risk.
4. **Governance Monitoring** — track board changes, filing compliance, and company status.

---

## 8. Functional Requirements

| ID | Requirement |
| --- | --- |
| FR1 | Ingest data from Companies House, Yahoo Finance, and Yahoo Finance News into the Bronze layer. |
| FR2 | Store data across three layers: Bronze (raw), Silver (cleaned), Gold (analytics-ready). |
| FR3 | Clean and normalize raw data: flatten nested JSON, parse dates, remove duplicates, handle nulls/NaN. |
| FR4 | Resolve company identity across sources by mapping Companies House numbers to Yahoo Finance tickers. |
| FR5 | Compute governance, market, financial, and news indicators and a composite company risk score. |

---

## 9. Non-Functional Requirements

| Attribute | Expectation |
| --- | --- |
| Scalability | Pipeline is parameterized; extendable beyond 20 companies. |
| Reliability | Repeatable, idempotent loads via SCD Type 1 MERGE. |
| Maintainability | Modular, layer-separated notebooks with shared helper functions. |
| Data Quality | Null/NaN handling, deduplication on business keys, score bounds (0–100). |
| Performance | Batch processing on Spark; long-format financial tables for efficient querying. |

---

## 10. Success Criteria

The project is successful when:

- Data is ingested from all three sources into Bronze.
- Data is cleaned and conformed into the Silver layer.
- Company ↔ ticker entity resolution is achieved.
- The Gold layer produces a per-company risk score in the range 0–100 with a risk band.
- Companies can be compared and ranked by risk.

---

## 11. Deliverables

- End-to-end Bronze → Silver → Gold data pipeline (Databricks notebooks).
- Structured datasets at each layer (Delta tables in Unity Catalog).
- Company-to-ticker mapping (`dim_company`).
- A four-pillar risk scoring model (`fact_company_risk`).
- Documentation: Business Requirements, Data Contract, System Architecture.

---

## 12. Risk Scoring Model (Summary)

The composite risk score is a weighted blend of four independent pillars, each normalized to 0–100 across the company universe (higher = higher risk):

| Pillar | Source tables | Example signals | Weight |
| --- | --- | --- | --- |
| Financial health | `yf_financials` | leverage, liquidity, margin, revenue growth | 0.35 |
| Market risk | `yf_stock`, `yf_info` | annualised volatility, max drawdown, beta | 0.30 |
| Governance | `ch_overview`, `ch_people`, `ch_filing_history` | company age, director churn, filing compliance | 0.25 |
| News signal | `yf_news` | article volume, basic sentiment | 0.10 |

```
risk_score = 0.35·financial + 0.30·market + 0.25·governance + 0.10·news
risk_band  = High / Medium / Low
```

The weights reflect data confidence: financial and market signals are strongest and best-populated; news is the lightest pillar and is weighted accordingly.

---

## 13. Future Enhancements

- Real-time / streaming ingestion.
- Machine-learning-based risk prediction.
- NLP-based sentiment analysis (replacing the lexicon approach).
- BI dashboard integration (Power BI / Tableau).
- Sector-relative scoring and expansion beyond the UK universe.
