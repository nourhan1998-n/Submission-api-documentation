# 🧩 Design Patterns in Submission API

This document explains the **design patterns** applied inside the **submission-api** and how they structure the flow of KYB request handling.  

---

## 🔹 CQRS (Command Query Responsibility Segregation)
- **Purpose**: Separate read and write operations for clarity and scalability.  
- **Usage in submission-api**:  
  - **Commands** → Create, amend, approve KYB requests.  
  - **Queries** → Fetch request status, list requests.  
- **Benefit**: Clear separation makes it easier to optimize queries and handle complex write logic independently.  

---

## 🔹 Chain of Responsibility
- **Purpose**: Pass a request through a chain of handlers where each one may process it or forward it.  
- **Usage in submission-api**:  
  - Applied in **validation pipeline**.  
  - Example: 
- **Benefit**: 

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
