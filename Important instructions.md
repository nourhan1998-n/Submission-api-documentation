# Important Instructions
1. [Adding Functionality to Existing Submission Engine](#1-adding-functionality-to-existing-submission-engine)  
2. [Adding APIs in RequestController](#2-adding-apis-in-requestcontroller)  

---

## 1. Adding Functionality to Existing Submission Engine

When extending existing functionality (e.g., adding new validation, transformation, or request action), developers **must follow the wrapping flow**.

### Wrapping Flow
```mermaid
flowchart LR
    A[Controller] --> B[Service Layer]
    B --> C[Submission Engine]
    C --> D[Validation & Rules]
    D --> E[Persistence Layer]
    E --> F[Response]
Key Guidelines
Always wrap new functionality in the existing flow rather than bypassing it.

Use the engine-provided services (e.g., request creation, update, submission, amendment, retrieval).

Plug into the validation, schema, and rule engine layers where necessary instead of duplicating logic.

Maintain auditability by ensuring all new functionality produces logs/events.

Ensure backward compatibility by not breaking existing flows.

## 2. Adding APIs in RequestController
When creating new endpoints in the RequestController, you must follow strict conventions because of the request filter applied at the controller level.

Path Structure Rule
All endpoints under /requests must contain the request reference immediately after /requests/.

✅ Correct Example:
PUT /requests/{reference}/amendments/resolve

❌ Incorrect Example:
PUT /requests/amendments/{reference}/resolve

Why This Rule Exists
A request filter inspects the path of each API.

The filter uses the request reference from the path to categorize and authorize requests.

If the reference is not placed immediately after /requests/, the filter will fail, and the request will be rejected.

Best Practices for Developers
Consistency – Follow existing naming and structure patterns for all APIs.

Validation First – Ensure schema validation is triggered via the engine, not manually in the controller.

Reusability – Extract shared logic into services; controllers should remain thin.

Documentation – Update API documentation (.md files) when new functionality or endpoints are added.

Testing – Add unit and integration tests for both success and failure flows.

Diagrams
Wrapping Flow

```mermaid
classDiagram
    %% Interfaces
    class BehaviorConstants {
        <<interface>>
        +<<attributes>>
    }

    class KybCategoryBehavior {
        <<interface>>
    }

    %% Implementations
    class KybAmendBehavior {
    }

    %% Abstract and concrete classes
    class AbstractCommandServiceDecorator {
        <<abstract>>
    }

    class AmendDecoratorService {
    }

    class AmendCommandWrapper {
    }

    class AmendRequestCommand {
    }

    %% Relationships
    BehaviorConstants <.. AmendDecoratorService : uses
    KybCategoryBehavior <|.. KybAmendBehavior : implements
    AbstractCommandServiceDecorator <|-- AmendDecoratorService : extends
    KybCategoryBehavior <.. AbstractCommandServiceDecorator : injected
    AmendDecoratorService --> AmendCommandWrapper : injected
    AmendCommandWrapper <|-- AmendRequestCommand : extends
