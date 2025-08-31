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

## 🔹 Command Design Pattern
- **Purpose**: Encapsulate a request (action) as an object.  
- **Usage in submission-api**:  
  - Each user action (submit request, upload document, rollback amendment) is a `Command`.  
  - Executed by a `CommandHandler`.  
- **Benefit**: Adds flexibility to queue, log, or rollback commands.  

---

## 🔹 Chain of Responsibility
- **Purpose**: Pass a request through a chain of handlers where each one may process it or forward it.  
- **Usage in submission-api**:  
  - Applied in **validation pipeline**.  
  - Example: SchemaValidation → BusinessValidation → Transformation.  
- **Benefit**: Decouples validation rules, allows adding new steps without changing the flow.  

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

---

## 🔹 Flow of a KYB Request
1. **User submits request** → REST Controller.  
2. **Command object** created and passed to **CommandHandler**.  
3. **Chain of Responsibility** executes schema validation → business validation → JOLT transformations.  
4. **Template method** defines persistence flow.  
5. Request persisted via **CustomPersistence** → `kyc-api`.  
6. **CQRS** ensures queries are separated from commands for efficient retrieval.  

---

## 🔹 Diagrams (placeholders)
- **Sequence diagram**: User → Submission API → Command → Validation Chain → Template Flow → KYC API → DB.  
- **Class diagram**: Command + Handler, ValidationChain, Template base class.  
