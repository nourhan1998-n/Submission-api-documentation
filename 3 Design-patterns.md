# 🧩 Design Patterns in Submission API

This document explains the **design patterns** applied inside the **submission-api** and how they structure the flow of KYB request handling.  

---

## 🔹 CQRS (Command Query Responsibility Segregation)
### **Purpose**: Separate read and write operations for clarity and scalability.  
### **Usage in submission-api**:  
  - **Commands** → Create, amend, approve KYB requests.  
  - **Queries** → Fetch request status, list requests.  
### **Benefit**: Clear separation makes it easier to optimize queries and handle complex write logic independently.  

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

## 🔹 Template Method Pattern
- **Purpose**: Define the skeleton of an algorithm in a base class, but let subclasses override specific steps.  
- **Usage in submission-api**:  
  - Request processing templates (e.g., “ProcessRequestTemplate”) with steps like:  
    1. Validate input  
    2. Apply business rules  
    3. Persist request  
    4. Notify  
  - Subclasses override validation or persistence depending on request type.  
- **Benefit**: Provides a standard flow while keeping extensibility.  
