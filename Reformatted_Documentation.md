# Submission & KYC System – Full Documentation

# 📑 Table of Contents

1. [System Architecture](#system-architecture)
   - [Overview](#overview)
   - [Data Sources](#data-sources)
   - [Contract Layer](#contract-layer)
   - [Architecture Diagram](#architecture-diagram)

2. [Database Model](#📘-database-model)
   - [Entity Relationship Highlights](#entity-relationship-highlights)
   - [Request & Processing](#request--processing)
   - [KYC Entities](#kyc-entities)
   - [Documents & Identity](#documents--identity)
   - [Workflow](#workflow)
   - [IAM & Security](#iam--security)
   - [Audit & Scheduled Tasks](#audit--scheduled-tasks)

3. [Design Patterns Used](#🧩-design-patterns)
   - [Chain of Responsibility](#chain-of-responsibility)
   - [Strategy Pattern](#strategy-pattern)
   - [Template Method Pattern](#template-method-pattern)
   - [CQRS Pattern](#cqrs-pattern)

4. [Validations](#✅-validations)
   - [Schema Validation](#1-schema-validation)
   - [Business Validation](#2-business-validation)
   - [JOLT Transformations](#3-transformations-with-jolt)

5. [Internal APIs](#🛠️-internal-apis)

6. [Public APIs](#🌐-public-apis)
   - [Update Request](#update-request)
   - [Update Request Partially](#update-request-partially)
   - [Amendment Resolution Flow](#validate--resolve-amend-errors-flow)

7. [Important Instructions](#📌-important-instructions)
   - [Extending Submission Engine](#1-adding-functionality-to-existing-submission-engine)
   - [Adding APIs to RequestController](#2-adding-apis-in-requestcontroller)


---

# 🏗️ System Architecture

## Overview
- **submission-api** library handles submission requests.
- **kyc-api** application manages KYC/KYB business logic, workflows and uses submission-api library.

## Data Sources
1. **Library Data Source** – stores submission requests (KYC).
2. **Application Data Source** – stores customer, KYC, business, documents, workflows (OCID).

## Contract Layer
Integration is via:
- **CustomPersistence Interface**
- **CustomRequestDocument Interface**

## Architecture Diagram
<img width="1564" height="791" alt="arch" src="https://github.com/user-attachments/assets/83c77a59-8c21-407c-9a3a-49b9898cb9bf" />


---

# 📘 Database Model

<img width="1048" height="1184" alt="db" src="https://github.com/user-attachments/assets/3e2761d8-14ce-48dc-a197-092336b0e11e" />
<p align="center"><i>Entities in red area are related to submission api datasource</i></p>

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
<img width="1664" height="752" alt="update" src="https://github.com/user-attachments/assets/2a676289-1695-440e-9879-181b43011137" />

---

## Update Request Partially
```
PUT /requests/{reference}/amendments/resolve
```
`isPatch = true`  
<img width="1664" height="752" alt="partially" src="https://github.com/user-attachments/assets/fbdb70d8-ce61-4137-9bf9-6cd1f282a55c" />

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
When extending existing functionality (e.g., adding new validation, transformation, or request action), developers **must follow the wrapping flow**.
<img width="1742" height="439" alt="f1" src="https://github.com/user-attachments/assets/f662f5fc-ba89-4cb5-aaeb-22d166a07995" />


### Why This Flow Exists
This design ensures that developers can easily **extend the amend process** by:  
- Adding new logic in the **pre-execute** phase (before the original amend flow runs).  
- Adding new logic in the **post-execute** phase (after the original amend flow finishes).  

In short, the flow provides a **decorator + wrapper pattern** that lets you inject custom functionality around the existing amend process **without breaking the original core logic**. 

---

### Example in Adding Functionality to Amend Flow

1. **AmendRequestCommand** defines the base command.  
2. **AmendCommandWrapper** extends it to add amend-specific logic.  
3. **AmendDecoratorService** (a service decorator) is injected into the wrapper to provide amend behaviors.  
4. **AmendDecoratorService** extends `AbstractCommandServiceDecorator`, which requires a `KybCategoryBehavior`.  
5. That **KybCategoryBehavior** is implemented by `KybAmendBehavior` (runtime logic).  
6. **AmendDecoratorService** also uses `BehaviorConstants` to standardize logic.

<img width="1812" height="692" alt="f2" src="https://github.com/user-attachments/assets/ed7302d3-a5b7-4c6e-9b69-37530500b10b" />
   
#### The flow wires together commands and decorators so that request handling can be extended dynamically (via decorator) and configured by category-specific behaviors (via injected behavior implementations).


### Key Guidelines
- Always wrap new functionality in the existing flow rather than bypassing it.  
- Use the engine-provided services (e.g., request creation, update, submission, amendment, retrieval).  
- Plug into the validation, schema, and rule engine layers where necessary instead of duplicating logic.  
- Maintain auditability by ensuring all new functionality produces logs/events.  
- Ensure backward compatibility by not breaking existing flows.  

---

## 2. Adding APIs in RequestController

When creating new endpoints in the `RequestController`, you must follow strict conventions because of the request filter applied at the controller level.

### Path Structure Rule
All endpoints under `/requests` must contain the request reference immediately after `/requests/`.

### Why This Rule Exists
- A request filter inspects the path of each API.  
- The filter uses the request reference from the path to categorize and authorize requests.  
- If the reference is not placed immediately after `/requests/`, the filter will fail, and the request will be rejected.  

### Path Rule (Critical)
🟩 Correct  
```
/requests/{reference}/amendments/resolve
```

🟥 Incorrect  
```
/requests/amendments/{reference}/resolve
```

---

