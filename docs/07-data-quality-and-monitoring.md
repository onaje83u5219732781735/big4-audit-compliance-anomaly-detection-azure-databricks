# 07 – Data Quality & Monitoring

## DQ dimensions

- **Completeness:** mandatory fields present.
- **Validity:** types and ranges valid.
- **Accuracy (proxy):** checks against reference data.
- **Consistency:** no conflicting values across sources.
- **Timeliness:** events arrive within expected time window.

## DQ checks

- Null and range checks on `amount`, `currency`, `country`, `mcc_code`.
- Referential checks against KYC, lists, and limits.
- Anomaly thresholds on volume (unexpected spikes or drops).

## Monitoring

- DQ results stored in `dq_results`.
- Aggregated metrics sent to Azure Monitor / Log Analytics.
- Alerts configured for:
  - DQ pass rate < 95% on critical checks.
  - DLQ volume exceeding normal ranges.
