# 05 – Data Models

This document lists the key Delta tables and their roles.

## 1. transactions_bronze

- **Grain:** One record per event from Event Hubs.
- **Partitioning:** `event_date` (DATE).
- **Usage:** Raw archive, replay source.

## 2. transactions_silver

- **Grain:** One validated and enriched transaction.
- **Partitioning:** `event_date`, `region`.
- **Columns:** Core transaction fields, enriched KYC/limits, derived behaviour features.

## 3. risk_scores_gold

- **Grain:** One scoring result per transaction.
- **Partitioning:** `event_date`, `risk_band`.
- **Columns:** transaction keys, risk score, risk band, model version, pipeline run ID, DQ status.

## 4. anomaly_alerts

- **Grain:** One row per high-risk transaction.
- **Usage:** Alerts and case-management integration.

## 5. dq_results

- **Grain:** One row per DQ check per batch / micro-batch.
- **Usage:** Evidence for audit and SLO monitoring.

## 6. regulatory_snapshots

- **Grain:** One row per transaction per snapshot date.
- **Usage:** Audit-ready, immutable snapshots for regulators and internal audit.
