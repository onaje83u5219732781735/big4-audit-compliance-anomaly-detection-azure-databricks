# 06 – ML Pipeline & Model Governance

This document expands on ML training, deployment, and governance.

## Training pipeline

- Runs as a **Batch Pipeline (ELT)**.
- Builds training datasets from `transactions_silver` and label tables.
- Stores feature matrices and labels in `ml_features_training`.

## Model lifecycle

1. **Develop** candidate models in Databricks.
2. **Track** experiments with MLflow (params, metrics, artifacts).
3. **Register** best models in MLflow Model Registry.
4. **Approve** models via governance process (risk, compliance, model validation).
5. **Promote** to `Production` when approved.
6. **Monitor** performance and drift; retrain as necessary.

## Governance

- Document each model's:
  - Intended use.
  - Limitations and known risks.
  - Training data coverage and assumptions.
- Maintain strict separation of dev, test, and prod environments.
