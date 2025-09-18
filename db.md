## 1. Core Service and Categories

### SERVICE
High-level service definition.  
**Fields:** ID, CODE (unique), REF_PREFIX (unique), STATUS (EXIST/DELETED), TERMS_AND_CONDITIONS.  
**Relations:** Related to CATEGORY and REQUEST.

### CATEGORY
Service categories (hierarchical, via `PARENT_ID`).  
**Fields:** ID, CODE, NAME_AR, NAME_EN, STATUS.  
**Relations:**  
- Belongs to SERVICE.  
- Connected with SCHEMA_REGISTRY.  
- Linked with CATEGORY_DOCUMENT & CATEGORY_FEE.  
- Linked to REQUEST.

---

## 2. Request & Processing

### REQUEST
Represents a client’s application.  
**Fields:** ID, REFERENCE (unique), STATUS, TYPE, CATEGORY_ID, SERVICE_ID.  
**Relations:**  
- Linked to CATEGORY & SERVICE.  
- Has many REQUEST_DOCUMENT, ERROR, REQUEST_ALLOWED_AUTHORS, REQUEST_INTEGRATION.  
- Referenced by KYC.

### REQUEST_DOCUMENT
Mapping between requests and uploaded documents.  
**Relations:** Links REQUEST → DOCUMENT → DOCUMENT_TYPE.  
Cascade deletes on DOCUMENT and DOCUMENT_TYPE.

### ERROR
Stores validation/business rule errors.  
**Fields:** ATTRIBUTE_PATH, MESSAGE, ERROR_TYPE (VALIDATION/BUSINESS/AMENDMENT), IS_RESOLVED.

### REQUEST_INTEGRATION
Tracks request-related external integrations.  
**Fields:** INTEGRATION_TYPE, PAYLOAD, TRANSFORMED_PAYLOAD, RESPONSE.

### REQUEST_ALLOWED_AUTHORS
Many-to-many mapping of who can author a request.

### REQUEST_DATA
Stores request payload or additional CLOB data.

---

## 3. KYC Entities

### CUSTOMER
High-level entity (organization or individual).  
**Fields:** ID, NAME, CODE, TRADE_LICENSE_NO, TYPE, CUSTOMER_ID.

### KYC
Central compliance record.  
**Fields:** STATUS, RISK_LEVEL, RISK_SCORE, CUSTOMER_ID, REQUEST_ID, KYC_STATUS, REVIEW_STATUS.  
**Relations:**  
- One KYC per REQUEST.  
- Linked to CUSTOMER.  
- Has BUSINESS, INDIVIDUAL.

### BUSINESS
Company details.  
**Fields:** COMPANY_NAME, TRADE_NAME, TRADE_LICENSE, COUNTRY_OF_INCORPORATION, ENTITY_STATUS, ENTITY_SUB_TYPE, BUSINESS_RELATION_TYPE.  
**Relations:** Linked to KYC.  
- Has many: BUSINESS_DOCUMENT, CONTACT, PARTNER, SHAREHOLDER, BUSINESS_INDIVIDUAL.

### INDIVIDUAL
Person details.  
**Fields:** NAME, NATIONALITY, IDENTIFICATION_NUMBER, EMAIL, POLITICALLY_EXPOSED, VIP, ID_VERIFICATION_STATUS.  
**Relations:** Linked to KYC.  
- Has many INDIVIDUAL_DOCUMENT, IDENTITY_TYPE.

### SHAREHOLDER
Ownership details.  
**Fields:** TYPE, IDENTIFIER, PERCENTAGE.  
**Relations:** Linked to BUSINESS.  
- Has many SHAREHOLDER_DOCUMENT.

### PARTNER
Business partners.  
**Relations:** Linked to BUSINESS.

### CONTACT
Contact & address information for businesses.  
**Relations:** Linked to BUSINESS.

---

## 4. Documents & Identity

### DOCUMENT_TYPE
Defines available document types (e.g., Passport, Trade License).  
**Fields:** ID, CODE (unique), NAME_AR, NAME_EN, STATUS (EXIST/DELETED), DOCUMENT_SCOPE.  
**Indexes:** By STATUS, DOCUMENT_SCOPE.  
**Relations:** Used in CATEGORY_DOCUMENT, REQUEST_DOCUMENT.

### DOCUMENT
General uploaded documents.  
**Fields:** NAME, DOCUMENT_SIZE, PATH, DOCUMENT_NUMBER, EXPIRY, CODE.  
**Relations:** Linked via REQUEST_DOCUMENT, BUSINESS_DOCUMENT, INDIVIDUAL_DOCUMENT, SHAREHOLDER_DOCUMENT.

### IDENTITY_TYPE
Identity types for individuals.  
**Fields:** ISSUING_DATE, EXPIRY_DATE.  
**Relations:** Linked to INDIVIDUAL.

### IDENTITY_DOCUMENT
Links DOCUMENT to IDENTITY_TYPE.

### BUSINESS_DOCUMENT / INDIVIDUAL_DOCUMENT / SHAREHOLDER_DOCUMENT
Link entities to their uploaded DOCUMENTs.

---

## 5. Workflow (State Machine)

### STATE, TRANSITION, ACTION, GUARD
Define lifecycle of requests.  
- **STATE**: supports entry/exit actions.  
- **TRANSITION**: connects states with optional guards.  
- **ACTION**: executable logic bound to states or transitions.  
- **GUARD**: conditional checks.

### STATE_MACHINE
Holds current machine states.

### Supporting Tables
- STATE_ENTRY_ACTIONS  
- STATE_EXIT_ACTIONS  
- STATE_STATE_ACTIONS  
- TRANSITION_ACTIONS  
- DEFERRED_EVENTS

---

## 6. Fees

### FEE_TYPE
Defines fee codes and names.

### CATEGORY_FEE
Links categories with applicable fees.

---

## 7. Rules and Schema

### RULE_SET and RULE
Define business rules (code, status, file paths).  
- A rule belongs to a RULE_SET.

### RULE_AUD, RULE_SET_AUD
Auditing versions of rules and sets.

### SCHEMA_REGISTRY
Stores data schemas for validation.

### REV_INFO
Revision metadata for auditing.

---

## 8. IAM & Security

### IAM_ENTITY_TYPE / IAM_IDENTITY_TYPE
Define entity and identity categories.

### IAM_BLOCK_LIST
Blacklist of identities.  
**Fields:** ENTITY_TYPE_ID, IDENTITY_TYPE_ID, IDENTITY_VALUE, STATUS.

---

## 9. Audit, Config, Logs

### DYNAMIC_CONFIGURATION
Stores runtime configuration.  
**Fields:** CONFIG_KEY, CONFIG_VALUE.

### HTTP_REQUEST_LOG / HTTP_REQUEST_LOG_2 / HTTP_REQUEST_LOG_3
Store HTTP requests and responses for auditing.

### DATABASECHANGELOG / DATABASECHANGELOGLOCK
Liquibase metadata tables.

### DELAYED_TASK
Retryable async task with attempt counts and failure reasons.

### HTE_REV_INFO
Temporary revision info table.

---

## 10. Views

### KYC_REQUEST_VIEW
Provides reporting join across KYC, REQUEST, BUSINESS, INDIVIDUAL, CUSTOMER.  
Fields include JIRA_TICKET, ENTITY_SUB_TYPE, COMPANY_NAME, SUBMITTED_AT, KYC_STATUS, REVIEW_STATUS, REQUEST REFERENCE, SOURCE_CHANNEL.
