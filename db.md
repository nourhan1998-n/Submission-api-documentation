
## 1. Core Service and Categories

- **SERVICE**  
  Represents a business service. Includes code, name, status, and terms.  
  **Relations:**
  - One service → many categories (`CATEGORY.SERVICE_ID`).

- **CATEGORY**  
  Defines request categories within a service. Supports hierarchy (`PARENT_ID`).  
  **Relations:**
  - Linked to `SERVICE` and optionally to `SCHEMA_REGISTRY` for data validation.  
  - One category → many documents (`CATEGORY_DOCUMENT`).  
  - One category → many fees (`CATEGORY_FEE`).  
  - One category → many requests (`REQUEST`).

---

## 2. Request Lifecycle

- **REQUEST**  
  Represents a client submission or application. Has reference, type, status, comments.  
  **Relations:**
  - Belongs to a `CATEGORY` and a `SERVICE`.  
  - Linked with documents (`REQUEST_DOCUMENT`), integrations (`REQUEST_INTEGRATION`), and errors (`ERROR`).  
  - Can have allowed authors (`REQUEST_ALLOWED_AUTHORS`).  
  - Extended with `REQUEST_DATA`.

- **REQUEST_DOCUMENT**  
  Links a request to uploaded documents with scope.  
  **Relations:**  
  - References `REQUEST`, `DOCUMENT`, and `DOCUMENT_TYPE`.

- **REQUEST_INTEGRATION**  
  Stores external system integration attempts for a request (payloads, responses).

- **REQUEST_ALLOWED_AUTHORS**  
  Defines which authors can modify a given request.

- **ERROR**  
  Stores errors linked to requests (validation, business, amendment).

- **REQUEST_DATA**  
  Holds additional request-related data in CLOB.

---

## 3. Document Management

- **DOCUMENT_TYPE**  
  Defines types of documents (code, name, max uploads).  
  **Relations:** Used by `CATEGORY_DOCUMENT` and `REQUEST_DOCUMENT`.

- **DOCUMENT**  
  Represents uploaded files (path, type, size, code).  
  **Relations:** Linked to requests (`REQUEST_DOCUMENT`), businesses (`BUSINESS_DOCUMENT`), shareholders (`SHAREHOLDER_DOCUMENT`), individuals (`INDIVIDUAL_DOCUMENT`), and identity documents (`IDENTITY_DOCUMENT`).

- **CATEGORY_DOCUMENT**  
  Connects categories with required documents. Supports conditional requirements.

- **BUSINESS_DOCUMENT**, **SHAREHOLDER_DOCUMENT**, **INDIVIDUAL_DOCUMENT**, **IDENTITY_DOCUMENT**  
  Linking tables connecting entities to their documents.

---

## 4. Fees

- **FEE_TYPE**  
  Defines fee codes and names.

- **CATEGORY_FEE**  
  Links categories with applicable fees.

---

## 5. Rules and Schema

- **RULE_SET** and **RULE**  
  Define business rules (code, status, file paths).  
  - A rule belongs to a `RULE_SET`.

- **RULE_AUD**, **RULE_SET_AUD**  
  Auditing versions of rules and sets.

- **SCHEMA_REGISTRY**  
  Stores data schemas for validation.

- **REV_INFO**  
  Revision metadata for auditing.

---

## 6. Customer and KYC

- **CUSTOMER**  
  Basic customer entity (name, code, type, etc.).

- **KYC**  
  Know Your Customer records linked to a `CUSTOMER` and a `REQUEST`. Contains risk, status, review info.

- **INDIVIDUAL**  
  Represents personal details linked to a `KYC`.

- **BUSINESS**  
  Company-level details linked to `KYC`.

- **Relationships:**  
  - Business ↔ Individuals: via `BUSINESS_INDIVIDUAL`.  
  - Business ↔ Partners (`PARTNER`).  
  - Business ↔ Contacts (`CONTACT`).  
  - Business ↔ Shareholders (`SHAREHOLDER`).  

- **Shareholder** can have documents (`SHAREHOLDER_DOCUMENT`).

- **Identity**: `IDENTITY_TYPE` + `IDENTITY_DOCUMENT` for individuals.

- **KYC_REQUEST_VIEW**  
  A reporting view joining KYC, requests, businesses, individuals, and customers.

---

## 7. State Machine (Workflow Engine)

- **STATE**, **TRANSITION**, **ACTION**, **GUARD**  
  Define workflow states, transitions, and actions.

- **STATE_MACHINE**  
  Holds current machine states.

- **Supporting Tables:** `STATE_ENTRY_ACTIONS`, `STATE_EXIT_ACTIONS`, `STATE_STATE_ACTIONS`, `TRANSITION_ACTIONS`, `DEFERRED_EVENTS`.

---

## 8. IAM / Access Control

- **IAM_ENTITY_TYPE** and **IAM_IDENTITY_TYPE**  
  Define identity classification.

- **IAM_BLOCK_LIST**  
  Stores blocked identities (entity type + identity type + value).

---

## 9. Logging & Config

- **DYNAMIC_CONFIGURATION**  
  Stores dynamic key-value config.

- **HTTP_REQUEST_LOG / 2 / 3**  
  Stores HTTP requests and responses for auditing.

- **DATABASECHANGELOG / DATABASECHANGELOGLOCK**  
  Liquibase metadata tables.

- **DELAYED_TASK**  
  Stores tasks with retry mechanisms (failure reason, retry times).

- **HTE_REV_INFO**  
  Temporary revision info table.
"""
