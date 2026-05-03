# Data Contract – Company Risk Intelligence Platform

## 1. Purpose

This data contract defines the structure, rules, and expectations for datasets used in the Company Risk Intelligence Platform.

It ensures consistency across ingestion, transformation, and analytics layers.

---

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
