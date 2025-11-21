# SECURITY.md – Security & Access Model

This document describes the **security model** for the Compliance & Anomaly Detection platform on Azure + Databricks.

All details are **generic and sanitised**.

## 1. Data classification

- **Public:** Documentation and synthetic examples in this repo.
- **Internal:** Configuration metadata, job definitions.
- **Confidential:** Transaction data, KYC, risk scores, anomaly alerts.
- **Restricted:** Any data that could identify individuals directly (PII).

In this sanitized case study, all data is synthetic, but the architecture assumes **Confidential / Restricted** handling.

## 2. Identity & access management (IAM)

- Azure AD security groups for:
  - `DE-Platform-Admins`
  - `DE-Platform-Operators`
  - `DE-Compliance-Analysts`
  - `DE-ReadOnly-BI`
- Access controlled through:
  - Databricks workspace permissions.
  - ACLs on Delta tables and views.
  - Row‑Level Security (RLS) / Column‑Level Security (CLS) on Azure SQL / Synapse views.
- No direct access to raw storage for end users; access via curated views or Delta tables.

## 3. Network security

- All components deployed into **Azure Virtual Networks** with private endpoints:
  - Event Hubs private endpoints.
  - ADLS / Delta storage private endpoints.
  - Databricks with VNet injection.
- No public ingress to the data plane; access via VPN / ExpressRoute and bastion as required.
- Network Security Groups (NSGs) restrict traffic between subnets.

## 4. Encryption

- **At rest:** CMK‑backed encryption using Azure Key Vault–managed keys (CMEK) for ADLS and databases.
- **In transit:** TLS enforced for all communications.
- Keys rotated according to enterprise policy (e.g., 12 months).

## 5. Secrets management

- All connection strings, service principals, and sensitive configuration values stored in **Azure Key Vault**.
- Databricks uses **secret scopes** that reference Key Vault secrets.
- No secrets are stored in notebooks, jobs, or configuration files.

## 6. Data governance & catalog

- **Azure Purview** used for:
  - Automated scanning of ADLS, Delta tables, and SQL / Synapse.
  - Lineage between Event Hubs → Databricks → Delta → BI.
  - Tagging of PII fields and critical data elements.
- Data retention policies applied per table (e.g., 7 years for regulatory snapshots).

## 7. Auditing & logging

- All privileged actions (schema changes, access grants, notebook updates) logged to **Azure Monitor / Log Analytics**.
- Databricks audit logs captured and retained according to enterprise policy.
- Access to risk scores and anomaly decisions is logged for compliance.

## 8. Responsibilities

- **Security Architecture** – central security / cloud team.
- **Data Governance** – data governance office and compliance stakeholders.
- **Platform Operations** – DataOps / SRE team.

For a real implementation, replace placeholders with organisation‑specific roles and policies.
