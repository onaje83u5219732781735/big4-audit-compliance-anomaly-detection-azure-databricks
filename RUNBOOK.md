# RUNBOOK – Compliance & Anomaly Detection – Azure + Databricks

This RUNBOOK describes day‑to‑day operations for the **Streaming Pipeline (ETL)**, **Batch Pipeline (ELT)**, and **ML pipelines** used for compliance & anomaly detection.

The goal is to make **replay, backfill, and model rollback** predictable and auditable.

---

## 1. Components covered

- **Streaming Pipeline (ETL)** – Event Hubs → Databricks Structured Streaming → Delta (Bronze/Silver/Gold).
- **Batch Pipeline (ELT)** – ADF + Databricks jobs for reconciliation, snapshots, and compaction.
- **ML Training Pipeline** – Databricks jobs / notebooks + MLflow.
- **ML Serving** – production model loaded in streaming job via MLflow Model Registry.

---

## 2. Day‑to‑day checks (Morning checklist)

1. Confirm all Databricks jobs and Event Hubs consumers are **running & healthy**.
2. Verify **SLO dashboards**:
   - Streaming latency p95 < 60s.
   - Batch reconciliation completed before T+1 06:00 UTC.
   - DQ pass rate ≥ 95% on critical rules.
3. Check **DLQ volume**:
   - Ensure DLQ volume is within normal range.
   - Investigate spikes (schema drift, upstream incidents).
4. Review **error logs** in Azure Monitor / Log Analytics:
   - Authentication / authorization failures.
   - Event Hub throttling or partition imbalance.
5. Confirm **Power BI dashboards** refreshed successfully.

---

## 3. Standard incident types

- **INC‑S01 – Streaming job failure or lag**
- **INC‑B01 – Batch reconciliation pipeline failure**
- **INC‑DQ01 – Massive DQ failures**
- **INC‑ML01 – Model degradation / drift**
- **INC‑DATA01 – Corrupted partition or bad backfill**

Each incident should be logged with:

- Incident ID, timestamps.
- Impacted tables and date ranges.
- Root cause and remediation steps.
- Any replay / backfill / rollback performed.

---

## 4. Streaming replay (reprocess subset of data)

**Goal:** Re‑run the Streaming Pipeline (ETL) for a specific time window (e.g., one day) because of an issue such as a bug fix, model upgrade, or upstream outage.

### 4.1 Preconditions

- Root cause understood (bug fixed, model version validated).
- Capacity validated for replay (cluster size, Event Hubs retention, Delta I/O).

### 4.2 Steps – replay from Bronze

1. **Identify impacted range**
   - Determine impacted `event_date` or timestamp range.
   - Record this in the incident ticket.

2. **Freeze downstream consumers (if needed)**
   - Pause downstream loads or inform users about partial data during replay.

3. **Create temporary sandbox tables**
   - Create `risk_scores_gold_tmp` and `anomaly_alerts_tmp` for the affected range.

4. **Run replay job**
   - Launch a dedicated **Databricks batch job** that:
     - Reads from `transactions_bronze` for the impacted range.
     - Applies the streaming logic (validation, enrichment, ML scoring) in batch mode.
     - Writes results into `*_tmp` tables.

5. **Validate replay results**
   - Compare record counts and key aggregates between old vs. new data.
   - Validate DQ metrics and SLOs.

6. **Swap partitions**
   - For the impacted range:
     - Delete or archive partitions from `risk_scores_gold` and `anomaly_alerts`.
     - Insert partitions from the `*_tmp` tables.
   - Update `regulatory_snapshots` if required (run snapshot job again).

7. **Clean up**
   - Drop `*_tmp` tables (or archive them for a defined retention period).

---

## 5. Batch backfill (historical data)

**Goal:** Onboard new historical periods (e.g., last 2 years) or re‑process long ranges after model/rule changes.

### 5.1 Preconditions

- Backfill scope agreed (date range, business lines).
- Cost & runtime impact discussed with stakeholders.

### 5.2 Steps

1. **Load historical files**
   - Use **ADF pipelines** to load files into ADLS and Delta staging tables.

2. **Run backfill job**
   - Launch a parametrized **Databricks batch job**:
     - Reads from staging (raw) tables.
     - Reuses the standard pipeline transformations and ML model used in production.
     - Writes to `transactions_bronze`, `transactions_silver`, and `risk_scores_gold` for historical dates.

3. **Run reconciliation**
   - Execute batch reconciliation pipelines to compare against system-of-record data.

4. **Create historical snapshots**
   - Run the snapshot pipeline to generate `regulatory_snapshots` for historical periods.

5. **Document**
   - Record the backfill range, model version used, and any deviations from standard pipelines.

---

## 6. Model rollback

**Goal:** Quickly revert to a safe previous model when the production model exhibits issues (e.g., false positives/negatives spike, drift).

### 6.1 Preconditions

- New model causing measurable degradation (KPIs, feedback from compliance teams).
- Previous model version available in **MLflow Model Registry**.

### 6.2 Steps

1. **Identify fallback model**
   - In MLflow Model Registry, select the last known good model version (e.g., `version 12`).

2. **Update registry stages**
   - Demote the current Production model to `Staging` or `Archived`.
   - Promote the fallback model to `Production`.

3. **Restart streaming jobs**
   - Restart the Databricks streaming job(s) to ensure they load the new Production model version.

4. **Monitor after rollback**
   - Intensively monitor KPIs and DQ for the next 24–48 hours.
   - Confirm that performance has stabilised.

5. **Record rollback**
   - Document:
     - Old and new model versions.
     - Time of rollback.
     - Reason and observed impact.

6. **Optional replay**
   - If needed, perform **targeted replay** (Section 4) for the period where the faulty model was active, so risk scores align with the rolled‑back model.

---

## 7. DQ failure handling (INC‑DQ01)

1. Identify whether the surge in DQ failures is caused by:
   - Upstream schema changes.
   - New product types / values.
   - Incorrect configuration or rules.

2. Triage:
   - Use DLQ tables to inspect example records.
   - Validate whether actual data is bad vs. our rules being too strict.

3. Short‑term action:
   - If necessary, temporarily **relax non‑critical DQ rules** while maintaining hard checks (e.g., primary keys).

4. Long‑term action:
   - Update DQ rule configuration and add tests.
   - Consider schema evolution or versioning in `transactions.schema.json`.

---

## 8. ML degradation and drift (INC‑ML01)

1. Monitor for symptoms:
   - Sudden change in the distribution of `risk_band`.
   - Business feedback that alerts are noisy or missing.
   - Drift metrics triggered (population stability index, feature drift).

2. Analysis:
   - Compare new data distributions vs. training data.
   - Review model performance metrics on recent labelled samples.

3. Actions:
   - Trigger new training run with updated data.
   - Consider **champion–challenger** deployment:
     - Route a percentage of traffic to a challenger model.
     - Compare metrics before promotion.

4. If severe:
   - Perform **model rollback** (Section 6).

---

## 9. Disaster recovery (DR) – high level

- All critical Delta tables (Bronze/Silver/Gold, snapshots) are stored in geo‑redundant storage.  
- Databricks notebooks & jobs are stored in Git (not represented in this docs‑only repo).  
- In DR region:
  - Recreate clusters, jobs, and secrets.
  - Restore Delta tables and Purview catalog.
  - Point Event Hubs and ADF pipelines to the DR region as per DR playbook.

---

## 10. Contacts & ownership (example)

- **Product Owner:** Compliance & Anomaly Platform Lead.
- **Tech Owner / Architect:** Data Engineering Lead.
- **Run‑Ops:** DataOps / SRE team.
- **Model Owner:** Head of Model Risk / Data Science Lead.

(Replace placeholders with actual names and groups in a real implementation.)
