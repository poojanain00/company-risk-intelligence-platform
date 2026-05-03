# company-risk-intelligence-platform
An end-to-end data engineering platform that integrates company registry data, financial market data, and news signals to build a unified company intelligence and risk analytics system.
# Company Risk Intelligence Platform

## Overview

An end-to-end data engineering platform that integrates company registry data, financial market data, and financial news to generate unified company intelligence and risk analytics.

The system enables holistic evaluation of company performance by combining legal, financial, and news-based signals into a single structured data model.

---

## Problem Statement

Company-related data is fragmented across multiple systems:

- Legal registry data (Companies House)
- Financial market data (Yahoo Finance)
- External news and events

This fragmentation makes it difficult to assess company health, risk, and performance in a unified way.

---

## Solution

This platform builds a unified data pipeline that:

- Ingests data from multiple external APIs
- Cleans and standardizes datasets
- Resolves entity mismatches across sources
- Generates analytical and risk-based insights

---

## Architecture

The system follows a Medallion architecture:

- Bronze Layer → Raw ingestion
- Silver Layer → Cleaned and structured data
- Gold Layer → Business intelligence and risk scoring

See `architecture.md` for full system design.

---

## Data Sources

- Companies House API (company registry data)
- Yahoo Finance API (stock market data)
- Yahoo Finance News (financial news events)

---

## Key Features

- Multi-source data ingestion
- Entity resolution (company ↔ stock ticker mapping)
- Time-series financial analysis
- News-driven event detection
- Company risk scoring engine
- Modular and scalable pipeline design

---

## Business Use Cases

- Company risk assessment
- Financial performance tracking
- News impact analysis
- Cross-company comparison
- Market behavior correlation analysis

---

## Outputs

- Structured company intelligence dataset
- Stock performance metrics
- Governance stability indicators
- News sentiment and event tracking
- Overall company risk scores

---

## Tech Stack

- Python
- Databricks / Delta Lake
- Pandas
- yfinance API
- Companies House API

---

## Future Improvements

- Real-time streaming ingestion
- Machine learning-based risk prediction
- Interactive dashboards
- Expanded global company coverage

---

## Author

Data Engineering Group Project
