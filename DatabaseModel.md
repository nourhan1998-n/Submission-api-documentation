# 📘 Database Model Documentation

This document describes the **KYB/KYC database schema** used in the project.  
It explains the entities, their fields, and how they relate to each other.

---

## 🔹 Core Reference Tables

- **DOCUMENT_TYPE**
  - Defines available document types (e.g., Passport, Trade License).
  - Fields: `ID`, `CODE` (unique), `NAME_AR`, `NAME_EN`, `STATUS (EXIST/DELETED)`, `DOCUMENT_SCOPE`.
  - Indexes: By `STATUS`, `DOCUMENT_SCOPE`.
  - Used in: `CATEGORY_DOCUMENT`, `REQUEST_DOCUMENT`.

- **FEE_TYPE**
  - Defines types of fees.
  - Fields: `ID`, `CODE` (unique), `NAME_AR`, `NAME_EN`, `STATUS (EXIST/DELETED)`.
  - Used in: `CATEGORY_FEE`.

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

- **REV_INFO / RULE_AUD / RULE_SET_AUD**
  - Store historical changes for rules.

- **DYNAMIC_CONFIGURATION**
  - Stores runtime configuration (`CONFIG_KEY`, `CONFIG_VALUE`).

- **HTTP_REQUEST_LOG(_2, _3)**
  - Request/response logging.

- **DELAYED_TASK**
  - Retryable async task with attempt counts and failure reasons.
