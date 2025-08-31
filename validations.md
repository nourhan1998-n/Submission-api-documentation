# ✅ Validation Types in the System

This document explains the different **validation mechanisms** applied in the system.

---

## 1. Schema Validation
- Schema is passed from **kyc-api → submission-api**.  
- Ensures correct **attribute names** and **data types** in KYB form.  
- Validation performed **before request is persisted**.  

(diagram placeholder)

---

## 2. Business Validation
- Implemented using **Drools rules**.  
- Rules stored under resources in `kyc-api`.  
- Ensures **business logic compliance** (ownership %, country restrictions, etc.).  

(diagram placeholder)

---

## 3. Transformations with JOLT
- **JOLT** used for JSON transformations.  
- Applied **before and after validation** for request mapping.  

(diagram placeholder)
