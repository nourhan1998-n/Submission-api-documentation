# 🏗️ System Architecture: Submission API & KYC API

This document explains how **submission-api** and **kyc-api** connect and interact.
---

## 🔹 Overview
- **submission-api**
  - Acts as entry point for **KYB submission requests**.
  - Stores only **KYB request data** in its own database.
- **kyc-api**
  - Manages **all KYC operations** and account-related data.
  - Owns **business rules, validations, and workflows**.

---

## 🔹 Two Data Sources
1. **Submission API Data Source**
   - Stores KYB submission requests.  
2. **KYC API Data Source**
   - Stores IAM_BlockList, Contact, Individual, Partner, Customer, Business and others related to settelment & payment.  

---

## 🔹 Contract Layer
- Integration via **CustomPersistence Interface** & **CustomRequestDocument Interface**.  
- Defines how `submission-api` interacts with `kyc-api`.  
- Ensures **loose coupling** between modules.  

---

## 🔹 Architecture Diagram
<img width="1564" height="791" alt="image" src="https://github.com/user-attachments/assets/f55a40c3-fe24-4005-8e8c-7d344d15977f" />
