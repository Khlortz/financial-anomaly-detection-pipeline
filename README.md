# Financial Anomaly Detection Pipeline

> Automated, containerized data pipeline combining rule-based and ML-driven anomaly detection — from synthetic transaction generation to an executive-ready Power BI dashboard.

![Financial Anomaly Dashboard](financial_anomaly_dashboard.png)

[![Apache Airflow](https://img.shields.io/badge/Apache%20Airflow-017CEE?style=flat-square&logo=apacheairflow&logoColor=white)](#)
[![dbt](https://img.shields.io/badge/dbt-FF694B?style=flat-square&logo=dbt&logoColor=white)](#)
[![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white)](#)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=flat-square&logo=postgresql&logoColor=white)](#)
[![Supabase](https://img.shields.io/badge/Supabase-3ECF8E?style=flat-square&logo=supabase&logoColor=white)](#)
[![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white)](#)
[![scikit-learn](https://img.shields.io/badge/scikit--learn-F7931E?style=flat-square&logo=scikit-learn&logoColor=white)](#)
[![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=flat-square&logo=powerbi&logoColor=black)](#)

---

## Problem Statement

Finance teams manually review thousands of transactions every month to catch anomalies — duplicate vendor payments, unexplained spend spikes, and suspicious round-number invoices. This process is slow, inconsistent, and doesn't scale as transaction volume grows.

This project builds an **automated, orchestrated pipeline** that ingests transactions daily, applies SQL-based business rules, runs an independent machine learning model, and surfaces every flagged transaction in a single, reviewable dashboard — with no manual intervention required.

---

## Architecture

```
┌─────────────────┐     ┌──────────────┐     ┌─────────────┐     ┌──────────────────┐     ┌─────────────┐
│  Python (Faker)  │ --> │  PostgreSQL   │ --> │     dbt     │ --> │  Isolation Forest │ --> │  Power BI   │
│  Data Generator   │     │  (Supabase)   │     │ Transform   │     │   Anomaly Model    │     │  Dashboard  │
└─────────────────┘     └──────────────┘     └─────────────┘     └──────────────────┘     └─────────────┘
        ▲
        │
┌───────┴────────┐
│ Apache Airflow  │   Orchestrates the daily pipeline run (Dockerized)
│   (Docker)      │
└─────────────────┘
```

**Pipeline flow:**
1. **Airflow DAG** triggers daily on a `@daily` schedule
2. **Python task** generates realistic synthetic financial transactions (via Faker)
3. Data lands in **PostgreSQL (Supabase)** as the raw layer
4. **dbt** transforms raw data into a clean, tested mart layer — applying three SQL-based anomaly rules
5. **Isolation Forest** (scikit-learn) independently scores every transaction for anomalous behavior
6. **Power BI** dashboard surfaces flagged transactions, trends, and department/vendor breakdowns

---

## Key Engineering Decisions

### Dual-layer anomaly detection
Rather than relying on a single detection method, this pipeline runs **two independent systems** and compares their output:

| Layer | Method | Catches |
|---|---|---|
| Rule-based (dbt/SQL) | Deterministic business logic | Round numbers, spend spikes (5x dept avg), duplicate vendor+amount pairs |
| ML-based (Python) | Isolation Forest (unsupervised) | Multi-dimensional outliers across amount, vendor, department, payment type |

**Validation result:** The ML model independently caught **94% of round-number anomalies** and showed strong overlap with spend-spike flags — cross-validating that both detection layers are identifying genuine anomalies rather than noise.

### Why Airflow + Docker
Apache Airflow doesn't run natively on Windows, so the entire orchestration layer runs in **Docker containers** (webserver, scheduler, worker, triggerer, Postgres metadata DB, Redis) — mirroring how this would be deployed in a real production environment rather than a one-off local script.

### Why dbt
All transformation logic — cleaning, type casting, and the three anomaly-flagging rules — lives in version-controlled, tested SQL models rather than embedded in application code. This gives:
- **Data lineage** (full graph from raw → staging → mart)
- **Automated testing** (uniqueness, not-null constraints)
- **Auto-generated documentation**

---

## Results

| Metric | Value |
|---|---|
| Total transactions processed | 5,100 |
| Anomalies detected (ML model) | 255 (5.0%) |
| Total amount flagged | $1.6M |
| Round-number flag → ML overlap | 94% |
| Spend-spike flag → ML overlap | High overlap (77 matches) |

**Top flagged vendor:** Aguilar-Harris — surfaced consistently across both detection layers, warranting manual review.

---

## Dashboard

The Power BI dashboard provides:
- **KPI summary** — total transactions, anomalies detected, dollar amount flagged, anomaly rate
- **Segmentation views** — anomalies by payment type and department (donut charts)
- **Vendor risk ranking** — bar chart of vendors with the most flagged transactions
- **Monthly trend** — anomaly volume over time
- **Transaction-level detail table** — sortable by anomaly score, with all three rule-based flags visible alongside the ML score, for direct finance team action

![Dashboard Detail](financial_anomaly_dashboard.png)

---

## Pipeline Orchestration (Apache Airflow)

The DAG `financial_anomaly_pipeline` runs daily, with two tracked tasks:

![Airflow DAG Graph](airflow_dag_graph.png)

- `generate_transactions` — creates new synthetic transactions via Faker
- `load_to_postgres` — loads transactions into the Supabase raw layer

Tagged: `finance`, `dbt`, `anomaly-detection` · Owner: `riyaz` · Schedule: `@daily`

---

## Data Lineage (dbt)

![dbt Lineage Graph](dbt_lineage_graph.png)

```
raw.raw_transactions → stg_transactions → mart_transactions
```

All models are tested for `unique` and `not_null` constraints on key columns, with auto-generated documentation via `dbt docs generate`.

---

## Tech Stack

| Category | Tools |
|---|---|
| Orchestration | Apache Airflow (Dockerized) |
| Data Transformation | dbt Core |
| Database | PostgreSQL (Supabase) |
| Machine Learning | Python, scikit-learn (Isolation Forest), Pandas |
| Data Generation | Python (Faker) |
| Visualization | Power BI, DAX |
| Infrastructure | Docker, Docker Compose |

---

## What I'd Improve Next

- Add a containerized **dbt run + Python ML scoring** step directly inside the Airflow DAG (currently the ML scoring runs as a separate notebook step)
- Implement **SMOTE-style synthetic anomaly injection** with configurable severity for better model benchmarking
- Add **Slack/email alerting** via Airflow on high-severity anomaly detection
- Move the Power BI data source from CSV export to a **direct, certificate-validated PostgreSQL connection**

---

## Author

**Shaik Riyaz Ahmed**
Business Intelligence Engineer · Analytics Engineer · Data Engineer
[LinkedIn](https://www.linkedin.com/in/riyazshaik020501/) · [GitHub](https://github.com/Khlortz)
