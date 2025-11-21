# 09 – Lineage & Audit

## Lineage

- Azure Purview collects lineage from:
  - Event Hubs → Databricks → Delta tables.
  - Delta tables → Azure SQL / Synapse → Power BI.
- Column-level lineage tracked for critical fields (amount, risk_score, risk_band).

## Audit evidence

For each scoring run:

- `pipeline_run_id`
- model version and run ID
- counts and DQ metrics
- snapshot identifiers (date, version)

This information supports **model risk management** and regulatory audits.
