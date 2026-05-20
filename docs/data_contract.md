# 📜 DATA CONTRACT  
## Company Risk Intelligence Platform

---

# 📑 TABLE OF CONTENTS

1. Introduction  
2. Data Product Overview  
3. Data Ownership  
4. Source System Details  
5. Source Table Details  
6. Business Definitions  
7. Target (Gold Layer Design)  
8. Data Access & Security  
9. Confidentiality Classification  
10. Data Refresh & Latency  
11. Data Quality Rules  
12. Data Dictionary  
13. Lineage  
14. Orchestration Details  
15. Consumption Layer  

---

# 1. INTRODUCTION

The Company Risk Intelligence Platform data product provides a unified, analytics-ready dataset that integrates company registry data, financial market data, and financial news signals.

The goal is to enable a holistic view of company performance, governance, and risk by combining multiple external data sources into a structured lakehouse architecture using Databricks.

---

# 2. DATA PRODUCT OVERVIEW

```yaml
data_product: company_risk_intelligence_platform
domain: financial_risk_analytics
description: >
  This data product integrates company registry data, stock market data,
  and financial news into a unified analytical model for company risk
  scoring, performance monitoring, and business intelligence.
## 2. Entities

### 2.1 Company Entity

| Field | Type | Description | Rules |
|------|------|-------------|------|
| company_id | string | Unique identifier | Not null |
| company_name | string | Official name | Required |
| company_number | string | Companies House ID | Unique |
| status | string | Active / Dissolved | Required |
| incorporation_date | date | Company start date | Valid date |
| sic_code | string | Industry classification | Optional |
| address | string | Registered office | Optional |

---

### 2.2 Stock Entity

| Field | Type | Description | Rules |
|------|------|-------------|------|
| ticker | string | Yahoo Finance ticker | Required |
| date | date | Trading date | Required |
| open | float | Opening price | Required |
| close | float | Closing price | Required |
| volume | integer | Trading volume | Required |

---

### 2.3 News Entity

| Field | Type | Description | Rules |
|------|------|-------------|------|
| ticker | string | Related company ticker | Required |
| title | string | News headline | Required |
| publish_time | timestamp | Publication time | Required |
| publisher | string | Source | Optional |
| summary | string | News summary | Optional |

---

### 2.4 Officer Entity

| Field | Type | Description | Rules |
|------|------|-------------|------|
| company_number | string | Linked company | Required |
| officer_name | string | Person name | Required |
| role | string | Job role | Required |
| appointed_on | date | Start date | Optional |
| resigned_on | date | End date | Optional |

---

## 3. Data Quality Rules

- No duplicate company_number entries
- Stock data must have continuous date range
- News must include valid timestamp
- Null values allowed only in optional fields
- All timestamps must be standardized (UTC)

---

## 4. Versioning

- Schema changes must be documented
- Backward compatibility must be maintained where possible
