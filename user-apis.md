# 🌐 User APIs Documentation

This document describes the **public APIs** exposed to end-users (e.g., customer portal, mobile app).  

---

## 🔹 Overview
- Exposed by: `submission-api`
- Purpose: Allow customers to submit KYB requests, upload documents, and track status.

---

## 🔹 Endpoints (Examples)
- `POST /requests` → Submit a new KYB request.
- `POST /requests/{id}/documents` → Upload documents for a request.
- `GET /requests/{id}/status` → Check status of a request.

(👉 Fill in with actual endpoints, request/response examples)

---

## 🔹 Example Flows

### 1. Submit KYB Request
**Flow:**  
User → `submission-api` → `kyc-api` → Database  

(diagram placeholder)

### 2. Upload Document
**Flow:**  
User → `submission-api` → Store Document → Link to Request  

(diagram placeholder)

### 3. Track Status
**Flow:**  
User → `submission-api` → `kyc-api` → Return Request Status  

### 4. Update Request Partially API

The **Update Request Partially** API allows partial updates to a submitted KYB request.  
It is primarily used during the **amendment resolution process**, where a business request may require changes based on back-office feedback, additional documents, or corrections from the customer.

**PUT** `/requests/{reference}/amendments/resolve`

---
<img width="1681" height="741" alt="image" src="https://github.com/user-attachments/assets/d7088622-a74c-458c-889a-bd03ecccc48f" />

