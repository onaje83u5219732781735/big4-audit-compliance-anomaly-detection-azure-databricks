# ADR 0001 – Use Azure Event Hubs + Databricks + Delta Lake for Compliance & Anomaly Detection

## Context

A Big‑4 audit & consulting firm requires a scalable platform for near real‑time compliance monitoring and anomaly detection on transaction and audit‑log data.

Key requirements:

- High‑throughput Streaming Pipeline (ETL) for risk scoring transactions.
- Batch Pipeline (ELT) for reconciliations and regulatory snapshots.
- Strong governance, lineage, and auditability.
- Integration with ML pipelines for anomaly detection.

## Decision

Use the following core stack on Azure:

- **Ingestion:** Azure Event Hubs for streaming, Azure Data Factory for batch.
- **Processing:** Azure Databricks (Structured Streaming + batch jobs) with PySpark.
- **Storage:** Delta Lake on ADLS with Bronze/Silver/Gold layers.
- **Serving:** Azure SQL / Synapse views with Power BI dashboards.
- **ML:** Databricks ML + MLflow Model Registry.
- **Governance:** Azure Purview, Key Vault, IAM/RBAC, RLS/CLS.

## Consequences

- Provides strong support for both Streaming Pipelines (ETL) and Batch Pipelines (ELT).
- Aligns with modern lakehouse architecture patterns used in BFSI and audit/compliance workloads.
- Requires investment in Databricks and Purview expertise.
