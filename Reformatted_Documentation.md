# Submission & KYC System – Full Documentation

# 📑 Table of Contents

1. [System Architecture](#system-architecture)
   - [Overview](#overview)
   - [Data Sources](#data-sources)
   - [Contract Layer](#contract-layer)
   - [Architecture Diagram](#architecture-diagram)
2. [Database Model](#database-model)
   - [Entity Relationship Highlights](#entity-relationship-highlights)
   - [Request & Processing](#request--processing)
   - [KYC Entities](#kyc-entities)
   - [Documents & Identity](#documents--identity)
   - [Workflow](#workflow)
   - [IAM & Security](#iam--security)
   - [Audit & Scheduled Tasks](#audit--scheduled-tasks)
3. [Design Patterns Used](#design-patterns)
   - [Chain of Responsibility](#chain-of-responsibility)
   - [Strategy Pattern](#strategy-pattern)
   - [Template Method](#template-method-pattern)
   - [CQRS Pattern](#cqrs-pattern)
4. [Validations](#validations)
   - [Schema Validation](#1-schema-validation)
   - [Business Validation](#2-business-validation)
   - [JOLT Transformations](#3-transformations-with-jolt)
5. [Internal APIs](#internal-apis)
6. [Public APIs](#public-apis)
   - [Update Request](#update-request)
   - [Update Request Partially](#update-request-partially)
   - [Amendment Resolution Flow](#validate--resolve-amend-errors-flow)
7. [Important Instructions](#important-instructions)
   - [Extending Submission Engine](#1-adding-functionality-to-existing-feature-in-submission-engine)
   - [Adding APIs to RequestController](#2-adding-apis-in-requestcontroller)

---

# 🏗️ System Architecture

## Overview
- **submission-api** handles submission requests.
- **kyc-api** manages KYC/KYB business logic and workflows.

## Data Sources
1. **Submission Data Source** – stores submission requests.
2. **KYC Data Source** – stores customer, KYC, business, documents, workflows.

## Contract Layer
Integration is via:
- **CustomPersistence Interface**
- **CustomRequestDocument Interface**

## Architecture Diagram
*(Image placeholder)*

---

# 📘 Database Model

## Entity Relationship Highlights
```
SERVICE → CATEGORY → REQUEST → KYC → BUSINESS/INDIVIDUAL → DOCUMENTS
```

### Core Concepts
- Requests link customers, documents, errors, and integrations.

---

## Request & Processing

### REQUEST
Represents a submission. Relates to:
- REQUEST_DOCUMENT
- ERROR
- REQUEST_INTEGRATION
- REQUEST_ALLOWED_AUTHORS

### REQUEST_DOCUMENT
Maps:
```
REQUEST → DOCUMENT → DOCUMENT_TYPE
```

### ERROR
Stores validation, business, and amendment errors.

---

## KYC Entities

### CUSTOMER
High-level entity (individual/business).

### KYC
Maps:
```
REQUEST ↔ CUSTOMER
```

### BUSINESS
Company details (trade license, incorporation country, shareholders, partners).

### INDIVIDUAL
Identity (name, nationality, ID verification).

---

## Documents & Identity

### DOCUMENT_TYPE
Defines valid document types.

### DOCUMENT
Stores uploaded files.

### IDENTITY_TYPE
Stores personal ID data.

---

## Workflow
Tables controlling lifecycle:
- STATE  
- TRANSITION  
- ACTION  
- GUARD  

---

## IAM & Security

### IAM_BLOCK_LIST
Blacklist of identities.

---

## Audit & Scheduled Tasks

### SCHEDULED_TASK
Handles delayed (15‑minute window) amendment execution.

---

# 🧩 Design Patterns

## Chain of Responsibility
Processors executed in order:
1. GDRFA validation  
2. Establishment enrichment  
3. Customer block checks  
4. Publish to KYC  

## Strategy Pattern
Used in `FlowStrategy` for category‑based:
- Persistence strategies  
- Document strategies  

## Template Method Pattern
Defines skeleton of request workflows.

## CQRS Pattern
- Commands → create, amend, approve  
- Queries → fetch & list  

---

# ✅ Validations

## 1. Schema Validation
Ensures structure, keys, data types.

## 2. Business Validation
Handled via **Drools** in kyc-api.

## 3. Transformations with JOLT
Used pre- and post‑validation.

---

# 🛠️ Internal APIs

(`/internal/requests`)
| HTTP Method | Endpoint | Description | Notes |
| :--- | :--- | :--- | :--- |
| **POST** | `/` | Initialize new request with category code and request info | |
| **GET** | `/` | Get paginated list of requests with filtering | Supports query params: `queryString`, `category`, `pageSize`, `pageNumber`, `sortingField`, `isAsc` |
| **GET** | `/` | Generate file export of requests | Requires `Accept: application/octet-stream` header |
| **POST** | `/{reference}/amend` | Admin amendment of submitted requests | |
| **POST** | `/clone` | Clone an existing request | |
| **PATCH** | `/reset` | Reset a request to its initial state | |
| **GET** | `/{reference}/documents` | Get all documents for a request | |
| **GET** | `/{reference}/documents/{scope}` | Get documents by scope | |
| **GET** | `/{reference}/allowed-authors` | Get list of allowed authors for a request | |

---

# 🌐 Public APIs

(`/requests`)

| HTTP Method | Endpoint | Description |
| :--- | :--- | :--- |
| **GET** | `/my-requests` | Get current user's requests |
| **GET** | `/owning-entity/{reference}` | Get requests by owning entity reference |
| **GET** | `/{reference}` | Get request details by reference |
| **GET** | `/{reference}/basic-info` | Get basic info of a request |
| **PUT** | `/{reference}` | Update full request |
| **PUT** | `/{reference}/amendments/resolve` | Partially update request to resolve amendments |
| **POST** | `/{reference}/documents` | Upload documents for a request |
| **PUT** | `/{reference}/submit` | Submit a request |
| **GET** | `/{reference}/document-types/document-scope/{documentScope}` | Get required documents for a request with specific scope |
| **GET** | `/{reference}/document-types` | Get all required documents for a request |
| **GET** | `/{reference}/documents/{scope}` | Get uploaded documents for a request by scope |
| **DELETE** | `/{reference}/documents/{code}` | Delete a specific document from a request |

## Update Request
```
PUT /requests/{reference}
```
`isPatch = false`  
*(Diagram placeholder)*

---

## Update Request Partially
```
PUT /requests/{reference}/amendments/resolve
```
`isPatch = true`  
*(Diagram placeholder)*

---

## Validate & Resolve Amend Errors Flow
Explains:
1. Fetch unresolved errors  
2. Split into categories  
3. Resolve errors without identifiers  
4. Validate errors with identifiers  
5. Merge results  
6. Return final amendment errors  

---

# 📌 Important Instructions

## 1. Adding Functionality to Existing Submission Engine
Follow:
- pre‑execute  
- main  
- post‑execute  
flow.  
*(Diagram placeholder)*

Guidelines:
- Never bypass engine flow  
- Always use decorators  
- Maintain auditability  

---

## 2. Adding APIs in RequestController

### Path Rule (Critical)
🟩 Correct  
```
/requests/{reference}/amendments/resolve
```

🟥 Incorrect  
```
/requests/amendments/{reference}/resolve
```

Because controller filter extracts `{reference}` from path immediately after `/requests/`.

---

