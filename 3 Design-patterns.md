# 🧩 Design Patterns in Submission API

This document explains the **design patterns** applied inside the **submission-api** and how they structure the flow of KYB request handling.  
## 📑 Table of Contents
- [🔹 Chain of Responsibility](#-chain-of-responsibility)  
- [🔹 Strategy Pattern](#-strategy-pattern)  
- [🔹 Template Method Pattern](#-template-method-pattern)  
- [🔹 CQRS Pattern](#-cqrs-pattern)
  
---

## 🔹 Chain of Responsibility

### Overview
The **Chain of Responsibility (CoR)** is a behavioral design pattern that allows a request to be processed by a sequence of handlers.  
Each handler has its own responsibility and can either:
- Process the request fully or partially.
- Pass the request along to the next handler in the chain.

### Usage in the Submission Engine
In the **Submission Engine**, the Chain of Responsibility pattern is applied inside the **submission action**, which is triggered by the **submit request** process.  
We need to perform **four ordered steps before a request can be submitted**:  
1. Validate against GDRFA rules.  
2. Enrich with establishment data.  
3. Check customer restrictions or blockages.  
4. Publish the request to the KYC system.
   
##### Handler Interface
Defines the contract for all handlers:
- `handle` → processes the request.  
- `getOrder` → determines the handler’s position in the chain.  

##### Submission Action
- Holds a list of handlers.  
- Iterates through handlers in order.  
- Passes the same request to all handlers.  
- Each handler can process or enrich the request and update a shared context.  

##### Concrete Handlers
The application defines four handlers, each with a specific responsibility:

| Handler                       | Responsibility                               | Example Order |
|-------------------------------|----------------------------------------------|---------------|
| GDRFAHandler                  | Validates request against GDRFA rules        | 1             |
| EstablishmentDataHandler      | Enriches request with establishment data     | 2             |
| CustomerBlockageHandler       | Checks customer restrictions and blockages   | 3             |
| KycRequestPublishingHandler   | Publishes the request to the KYC system      | 4             |

### Summary
- **Handler Interface** defines responsibilities and execution order.  
- **Submission Action** executes all handlers in sequence on the request.  
- **Handlers** each implement a single responsibility in the submission process.  
- **Advantages** include separation of concerns, extensibility, and consistent processing order.  

---

## 🔹 Strategy Pattern

### Context of Use
In the **Submission Engine**, the **Strategy pattern** is applied within the `FlowStrategy` class.  
The `FlowStrategy` decides how a request should be processed based on the **category** (e.g., KYC, KYB).  

To achieve this, the class holds two maps:  
- **customPersistenceServiceMap** → selects the persistence strategy.  
- **customRequestDocumentServiceMap** → selects the document handling strategy.  

When a request is processed, the appropriate strategy is chosen dynamically at runtime according to the category passed in.

### Flow Strategy
- Acts as the **context** for strategy execution.  
- Relies on `customPersistenceServiceMap` and `customRequestDocumentServiceMap`.  
- Based on the request’s **category**, it picks the appropriate strategy from the maps.  
- Executes persistence and document-handling strategies in a consistent manner.  

### Benefits in This Context
- Centralizes flow handling in the `FlowStrategy` class while keeping actual logic separate.  
- Supports **category-specific behavior** without cluttering the core submission engine.  
- Easy to introduce new categories or services by simply adding new strategies.  
- Encourages modular, testable, and loosely coupled design.  

### Summary
- The **Strategy pattern** is used in the `FlowStrategy` class.  
- **Maps (customPersistenceServiceMap, customRequestDocumentServiceMap)** provide the correct strategy implementation based on category.  
- **Each category** (KYC, KYB, GVPAY, etc.) has its own persistence and document-handling strategies.  
- **Advantages**: separation of concerns, extensibility, flexibility, and maintainability.  

---

## 🔹 Template Method Pattern

### Purpose
The **Template Method pattern** is a behavioral design pattern that defines the **skeleton of an algorithm** in a base class, while allowing subclasses to override specific steps without changing the overall flow.  

This ensures that all implementations follow a **standard sequence of operations**, but remain flexible where customization is required.

### Usage in Submission API
The Template Method pattern is applied in **multiple flows** within the submission-api.  

Subclasses **override only the steps that differ** depending on the type of request or business rules involved, while the overall structure of the process remains consistent.

### Benefits in This Context
- Provides a **standardized flow** for processing requests across different categories.  
- Reduces duplication by centralizing common process steps.  
- Allows controlled **variation** where business logic differs.  
- Makes request processing easier to understand, extend, and maintain.
  
### Summary
The **Template Method pattern** is used in the submission-api to define standardized request-processing flows while allowing controlled variation in specific steps.  
This ensures both **consistency** across all request types and **flexibility** to adapt to category-specific requirements.

---

## 🔹 CQRS Pattern
### **Purpose**: Separate read and write operations for clarity and scalability.  
### **Usage in submission-api**:  
  - **Commands** → Create, amend, approve KYB requests.  
  - **Queries** → Fetch request status, list requests.  
### **Benefit**: Clear separation makes it easier to optimize queries and handle complex write logic independently.  



  
