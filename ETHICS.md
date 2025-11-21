# ETHICS.md – Sanitisation & Responsible Use

This repository is a **sanitised, docs‑only case study** of a Compliance & Anomaly Detection platform for a Big‑4 audit & consulting firm.

## 1. No real client data

- All described data, tables, and schemas are **synthetic**.
- Any resemblance to real clients, systems, or values is coincidental.
- The purpose is to show **patterns and architecture**, not to expose confidential information.

## 2. Responsible AI & model use

- Anomaly detection models should be:
  - Transparent and explainable to compliance and audit teams.
  - Tested for bias and unintended discrimination.
  - Governed by clear approval and monitoring processes.
- Model decisions must **support human analysts**, not replace them:
  - High‑risk alerts should be reviewed by trained staff.
  - Feedback loops should be used to refine models and rules.

## 3. Data privacy

- In production, only the **minimum necessary personal data** should be processed.
- Pseudonymisation or tokenisation should be used where possible.
- Access to PII should be tightly controlled and logged.

## 4. Limitations of this repo

- This repo does **not** contain end‑to‑end production code.
- It should not be treated as a drop‑in solution for regulatory reporting or compliance; additional design and validation is required for each organisation.

Use this project as a **learning and discussion artefact** for data engineering, ML pipelines, and governance patterns.
