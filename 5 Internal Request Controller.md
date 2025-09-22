# 🛠️ Admin APIs Documentation

This document describes the **internal APIs** used by the back-office system.  

---

## 🔹 Overview
- Exposed by: `kyc-api`
- Purpose: Allow internal staff to review, amend, approve, or reject KYB/KYC requests.

---

## 🔹 Endpoints (Examples)
- `GET /admin/requests/{id}` → View request details.
- `POST /admin/requests/{id}/amend` → Amend request.
- `POST /admin/requests/{id}/approve` → Approve request.
- `POST /admin/requests/{id}/reject` → Reject request.

(👉 Fill in with actual endpoints, request/response examples)

---

## 🔹 Example Flows

### 1. Review Request
Admin → `kyc-api` → Database  

(diagram placeholder)

### 2. Amend Request
Admin → `kyc-api` → Update Request → Revalidate  

(diagram placeholder)

### 3. Approve/Reject Request
Admin → `kyc-api` → Update Status → Notify User  

(diagram placeholder)
