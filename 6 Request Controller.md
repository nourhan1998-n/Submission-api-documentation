# 🌐 Request Controller Documentation

This document describes the **public APIs** exposed to end-users (e.g., customer portal, mobile app).  

---

## 🔹 Overview
- Exposed by: `submission-api`
- Purpose: Allow customers to submit/update KYB requests, upload documents, and track status.

---

## 🔹 Endpoints (Examples)
- `PUT /requests/{reference} ` → Update request.
- `PUT /requests/{reference}/amendments/resolve ` → Resolve request and update it if amended.

---

## 🔹 Flows  

### 1. Update Request

**PUT** `/requests/{reference}` isPatch=**No**
<img width="1664" height="752" alt="image" src="https://github.com/user-attachments/assets/1b545685-f64d-4c8c-a7e8-4880084b4254" />

### 2. Update Request Partially API

The **Update Request Partially** API allows partial updates to a submitted KYB request.  
It is primarily used during the **amendment resolution process**, where a business request may require changes based on back-office feedback, additional documents, or corrections from the customer.

**PUT** `/requests/{reference}/amendments/resolve` isPatch=**Yes**
<img width="1664" height="752" alt="image" src="https://github.com/user-attachments/assets/1b545685-f64d-4c8c-a7e8-4880084b4254" />

#### Validate & resolve amend errors Flow
This method validates and resolves amendment-related errors for a given request.  
It works in the following steps:
##### 1. Fetch unresolved errors
- Get all errors for the given request (`requestId`) where `isResolved = false`.
- Keep only errors that:
  - Have a non-null `elementIdentifier`, **or**
  - Match one of the `attributePaths` (exactly or as a prefix).

##### 2. Split errors into categories
- **Validation/Business Errors** → collect their `attributePath`s.
- **Amendment Errors** → separate for special handling.

##### 3. Handle amendment errors without element identifiers
- For each amendment error without an `elementIdentifier`:
  - If its `attributePath` is **not already covered** by validation/business errors **and**  
    the `customErrorServiceStrategy` says it should not be skipped,  
    → mark the error as **resolved**.
- Collect all such errors into a set (`errorsWithoutIdentifier`). 

##### 4. Handle amendment errors with element identifiers
- Send them to `customValidationService.validateAndResolveErrorsWithIdentifier`,  
  which validates and possibly resolves them.
- Collect results into `errorsWithIdentifier`. 

##### 5. Merge resolved errors
- Combine `errorsWithoutIdentifier` and `errorsWithIdentifier` into one list (`resolvedErrors`).
- Save all these resolved errors back to the database. 

##### 6. Return updated amendment errors
- Query the repository for **all amendment errors** of the request.
- Map them into domain models using `errorBoundedMonoMapper`.
- Return the mapped list.



