# 📘 Database Model Documentation

This document describes the **SUBMISSION-API database schema** used in the project.  

---
The **`kyc-api`** is the core service responsible for handling both **KYC (Know Your Customer)** and **KYB (Know Your Business)** processes.  

- The **`kyb` module** builds on top of the **submission-api**, which it uses as a library.  
- As a result, the system operates with **two data sources**:
  1. **Submission API Data Source** → Stores data related only to **KYB requests**.  
  2. **KYC API Data Source** → Stores all other **account operations and KYC data**.  

This separation ensures that **KYB request processing** is isolated in the submission-api layer, while **account-related operations** remain centralized in the kyc-api database.  

---
<img width="1048" height="1184" alt="KYC_RC_DEV2_EDITED" src="https://github.com/user-attachments/assets/a509a2e8-3872-4e38-b5d5-224bc76937d6" />

## 🔹 Core Reference Tables

- **SERVICE**
  - High-level service definition.
  - Fields: `ID`, `CODE` (unique), `REF_PREFIX` (unique), `STATUS (EXIST/DELETED)`, `TERMS_AND_CONDITIONS`.
  - Related to: `CATEGORY`, `REQUEST`.

- **CATEGORY**
  - Service categories (hierarchical, via `PARENT_ID`).
  - Fields: `ID`, `CODE`, `NAME_AR`, `NAME_EN`, `STATUS`.
  - Relationships:
    - Belongs to `SERVICE`.
    - Connected with `SCHEMA_REGISTRY`.
    - Linked with `CATEGORY_DOCUMENT` & `CATEGORY_FEE`.

---

## 🔹 Request & Processing

- **REQUEST**
  - Represents a client’s application.
  - Fields: `ID`, `REFERENCE` (unique), `STATUS`, `TYPE`, `CATEGORY_ID`, `SERVICE_ID`.
  - Relationships:
    - Linked to `CATEGORY` & `SERVICE`.
    - Has many: `REQUEST_DOCUMENT`, `ERROR`, `REQUEST_ALLOWED_AUTHORS`, `REQUEST_INTEGRATION`.
    - Referenced by `KYC`.

- **REQUEST_DOCUMENT**
  - Mapping between requests and uploaded documents.
  - Links: `REQUEST → DOCUMENT → DOCUMENT_TYPE`.
  - Cascade deletes on `DOCUMENT` and `DOCUMENT_TYPE`.

- **ERROR**
  - Stores validation/business rule errors.
  - Fields: `ATTRIBUTE_PATH`, `MESSAGE`, `ERROR_TYPE (VALIDATION/BUSINESS/AMENDMENT)`, `IS_RESOLVED`.

- **REQUEST_INTEGRATION**
  - Tracks request-related external integrations.
  - Fields: `INTEGRATION_TYPE`, `PAYLOAD`, `TRANSFORMED_PAYLOAD`, `RESPONSE`.

- **REQUEST_ALLOWED_AUTHORS**
  - Many-to-many mapping of who can author a request.

---

## 🔹 KYC Entities

- **CUSTOMER**
  - High-level entity (organization or individual).
  - Fields: `ID`, `NAME`, `CODE`, `TRADE_LICENSE_NO`, `TYPE`, `CUSTOMER_ID`.

- **KYC**
  - Central compliance record.
  - Fields: `STATUS`, `RISK_LEVEL`, `RISK_SCORE`, `CUSTOMER_ID`, `REQUEST_ID`, `KYC_STATUS`, `REVIEW_STATUS`.
  - Relationships:
    - One `KYC` per `REQUEST`.
    - Linked to `CUSTOMER`.
    - Has: `BUSINESS`, `INDIVIDUAL`.

- **BUSINESS**
  - Company details.
  - Fields: `COMPANY_NAME`, `TRADE_NAME`, `TRADE_LICENSE`, `COUNTRY_OF_INCORPORATION`, `ENTITY_STATUS`, `ENTITY_SUB_TYPE`, `BUSINESS_RELATION_TYPE`.
  - Linked to `KYC`.
  - Has many: `BUSINESS_DOCUMENT`, `CONTACT`, `PARTNER`, `SHAREHOLDER`, `BUSINESS_INDIVIDUAL`.

- **INDIVIDUAL**
  - Person details.
  - Fields: `NAME`, `NATIONALITY`, `IDENTIFICATION_NUMBER`, `EMAIL`, `POLITICALLY_EXPOSED`, `VIP`, `ID_VERIFICATION_STATUS`.
  - Linked to `KYC`.
  - Has many: `INDIVIDUAL_DOCUMENT`, `IDENTITY_TYPE`.

- **SHAREHOLDER**
  - Ownership details.
  - Fields: `TYPE`, `IDENTIFIER`, `PERCENTAGE`.
  - Linked to `BUSINESS`.
  - Has many: `SHAREHOLDER_DOCUMENT`.

- **PARTNER**
  - Business partners (linked to `BUSINESS`).

- **CONTACT**
  - Contact & address information for businesses.

---

## 🔹 Documents & Identity

- **DOCUMENT_TYPE**
  - Defines available document types (e.g., Passport, Trade License).
  - Fields: `ID`, `CODE` (unique), `NAME_AR`, `NAME_EN`, `STATUS (EXIST/DELETED)`, `DOCUMENT_SCOPE`.
  - Indexes: By `STATUS`, `DOCUMENT_SCOPE`.
  - Used in: `CATEGORY_DOCUMENT`, `REQUEST_DOCUMENT`.

- **DOCUMENT**
  - General uploaded documents.
  - Fields: `NAME`, `DOCUMENT_SIZE`, `PATH`, `DOCUMENT_NUMBER`, `EXPIRY`, `CODE`.
  - Linked via: `REQUEST_DOCUMENT`, `BUSINESS_DOCUMENT`, `INDIVIDUAL_DOCUMENT`, `SHAREHOLDER_DOCUMENT`.

- **IDENTITY_TYPE**
  - Identity types for individuals.
  - Fields: `ISSUING_DATE`, `EXPIRY_DATE`.
  - Linked to `INDIVIDUAL`.

- **IDENTITY_DOCUMENT**
  - Links `DOCUMENT` to `IDENTITY_TYPE`.

---

## 🔹 Workflow (State Machine)

- **STATE, TRANSITION, ACTION, GUARD**
  - Define lifecycle of requests.
  - `STATE` supports entry/exit actions.
  - `TRANSITION` connects states with optional guards.

---

## 🔹 IAM & Security

- **IAM_ENTITY_TYPE / IAM_IDENTITY_TYPE**
  - Define entity and identity categories.

- **IAM_BLOCK_LIST**
  - Blacklist of identities.
  - Fields: `ENTITY_TYPE_ID`, `IDENTITY_TYPE_ID`, `IDENTITY_VALUE`, `STATUS`.

---

## 🔹 Audit, Config, Logs

- **DYNAMIC_CONFIGURATION**
  - Stores runtime configuration (`CONFIG_KEY`, `CONFIG_VALUE`).

- **SCHEDULED_TASK**
  - Retryable async task with attempt counts and failure reasons.
