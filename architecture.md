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
   - Stores all other operations (customers, accounts, rules).  

---

## 🔹 Contract Layer
- Integration via **CustomPersistence Interface** & **CustomRequestDocument Interface**.  
- Defines how `submission-api` interacts with `kyc-api`.  
- Ensures **loose coupling** between modules.  

---

## 🔹 Architecture Diagram
<img width="1413" height="720" alt="image" src="https://github.com/user-attachments/assets/8a1c769a-88c8-4179-9f51-e27b4bd92cef" />
