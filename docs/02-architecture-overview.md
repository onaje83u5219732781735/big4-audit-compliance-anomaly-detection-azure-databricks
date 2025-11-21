# 02 – Architecture Overview

This platform combines:

- **Streaming Pipeline (ETL)** for near real‑time scoring and alerting.
- **Batch Pipeline (ELT)** for reconciliations, regulatory snapshots, and historical backfill.
- **ML pipelines** for training, registering, and serving anomaly detection models.

See the L2 diagram and sequence diagram in `README.md` for a visual overview.

Key design choices:

- Delta Lake Bronze/Silver/Gold layers for reliability and performance.
- ML integration via Databricks + MLflow with clear model governance.
- Strong governance using Purview, Key Vault, IAM/RBAC, and RLS/CLS.
