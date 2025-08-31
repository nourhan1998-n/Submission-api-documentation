# 🏗️ System Architecture: Submission API & KYC API

This document explains how **submission-api** and **kyc-api** connect and interact.

---

## 🔹 Overview
- **submission-api**
  - Acts as entry point for **KYB requests**.
  - Stores only **KYB request data** in its own database.
- **kyc-api**
  - Manages **all KYC operations** and account-related data.
  - Owns **business rules, validations, and workflows**.

---

## 🔹 Two Data Sources
1. **Submission API Data Source**
   - Stores KYB requests only.  
2. **KYC API Data Source**
   - Stores all other operations (KYC, customers, accounts, rules).  

---

## 🔹 Contract Layer
- Integration via **CustomPersistence Interface**.  
- Defines how `submission-api` interacts with `kyc-api`.  
- Ensures **loose coupling** between modules.  

---

## 🔹 Architecture Diagram (Placeholder)
submission-api → (CustomPersistence) → kyc-api → Databases
