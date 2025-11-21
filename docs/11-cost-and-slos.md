# 11 – Cost & SLOs

## Cost controls

- Use autoscaling clusters for Databricks streaming and batch jobs.
- Optimise Delta tables with partitioning and file compaction.
- Apply data retention and tiering policies (cool/archive for old snapshots).
- Configure Azure Budgets and alerts for spend thresholds.

## SLOs (example set)

- **Streaming latency:** p95 < 60s from Event Hubs to `risk_scores_gold`.
- **Batch reconciliation:** completed by T+1 06:00 UTC.
- **DQ pass rate:** ≥ 95% on critical checks.
- **Availability:** 99.9% for core Streaming Pipeline (ETL).

These are simulated SLOs for illustration and must be tuned for each real implementation.
