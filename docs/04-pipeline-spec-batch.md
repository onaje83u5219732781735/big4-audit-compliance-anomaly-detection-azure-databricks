# 04 – Pipeline Spec – Batch Pipeline (ELT)

## Purpose

Perform reconciliations, compaction, and regulatory snapshot creation.

## Sources

- System-of-record transaction files.
- Streaming outputs: `transactions_silver`, `risk_scores_gold`.

## Steps

1. **Load** system-of-record files to staging tables via ADF.
2. **Reconcile** counts and metrics with streaming outputs.
3. **Compact** and **optimize** Delta tables.
4. **Create snapshots** in `regulatory_snapshots`.
5. **Produce reports** and expose them through Azure SQL / Synapse and Power BI.

## Schedule

- Daily T+1 run for reconciliations and snapshots.
- Weekly / monthly runs for compaction and optimisation.
