# 🌐 User APIs Documentation

This document describes the **public APIs** exposed to end-users (e.g., customer portal, mobile app).  

---

## 🔹 Overview
- Exposed by: `submission-api`
- Purpose: Allow customers to submit/update KYB requests, upload documents, and track status.

---

## 🔹 Endpoints (Examples)
- `PUT /requests/{reference}/amendments/resolve ` → Resolve request and update it if amended.

---

## 🔹 Flows  

### 1. Update Request Partially API

The **Update Request Partially** API allows partial updates to a submitted KYB request.  
It is primarily used during the **amendment resolution process**, where a business request may require changes based on back-office feedback, additional documents, or corrections from the customer.

**PUT** `/requests/{reference}/amendments/resolve`
<img width="1664" height="752" alt="image" src="https://github.com/user-attachments/assets/1b545685-f64d-4c8c-a7e8-4880084b4254" />

#### Validate & resolve amend errors Flow
This method validates and resolves amendment-related errors for a given request.  
It works in the following steps:

1. **Fetch unresolved errors**  
   Retrieve all unresolved errors for the given request ID from the repository.
2. **Filter relevant errors**  
   Keep only the errors that:
   - Have a non-null element identifier, or  
   - Match one of the provided attribute paths (either exactly or as a prefix).
3. **Split errors into groups**  
   - **Validation/Business errors**: Errors that are not of type `AMENDMENT`.  
   - **Amendment errors**: Errors that are of type `AMENDMENT`.
4. **Process amendment errors without identifier**  
   - Ignore errors whose attribute path already exists in validation/business errors.  
   - For the rest:
     - Check with the error validation service if they should be skipped.  
     - If not skipped, mark them as resolved.  
   - Collect these errors in a separate set.
5. **Process amendment errors with identifier**  
   - Pass these errors to the custom validation service to validate and resolve them.
6. **Combine resolved errors**  
   Merge the sets of errors resolved with and without identifiers.
7. **Save resolved errors**  
   Persist the resolved errors back into the repository.
8. **Return amendment errors**  
   Retrieve all amendment errors for the request (still unresolved or newly created),  
   map them to the domain model, and return them.


