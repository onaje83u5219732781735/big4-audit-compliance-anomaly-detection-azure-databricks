# 03 – Pipeline Spec – Streaming Pipeline (ETL)

## Purpose

Ingest and process transaction events in near real time, producing risk scores and anomaly alerts.

## Sources

- Azure Event Hubs topic: `transactions-stream`

## Steps

1. **Ingest** events from Event Hubs into `transactions_bronze`.
2. **Validate** using `contracts/transactions.schema.json`.
3. **Run DQ checks** (nulls, ranges, referential checks).
4. **Enrich** with KYC, limits, watchlists.
5. **Score** with ML model from MLflow registry.
6. **Write** to:
   - `transactions_silver`
   - `risk_scores_gold`
   - `anomaly_alerts`
7. **Publish metrics** to Azure Monitor / Log Analytics.

## Output SLA

- p95 latency from Event Hubs to `risk_scores_gold` < 60 seconds.
