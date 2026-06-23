# KYB Review Status — Visual Diagrams

---

## A. KYB System Architecture

![KYB System Architecture](https://mermaid.ink/img/pako:eNqNVm1v2jAQ_iuWP7UV0EL3MmkfAgkEEsrLkrKHCTkkFokT-YOmKvrvu8RJKKUdQ3Lunnvu7mz7mUQ8IcQjnIk9T3ckjaKEKMXl54WG_MAZl5s_KfG-MKFSJqmYkmBNpUJnpnxN0oeYpxm_ZDSPYxZlKbyOmcw5NfbQD_eQiHSPmqUSnrIsn_ILTlNJIp5KdUVBLKliEuIslWLFJBcx5Q80WYi0FNqFZkKlakMkBz3M6oA2hLIbmrNfLKNM72P4P2bK0NxEXd7u3G4gF_6BizAQekYs7SJTMHM0CiS7ZSwuQAB4IZIJyXMFLxK2SnkBGaI5kWYRhQcWpTjJW2JPKA_Bi2i_QmPPk9rG3jN3kcJ-bBW8bR_xxATM_MqLlMqgfWAiYRm8tXe4N8KhUa_vbsiwc31aVjf6OU1zU86-dCKpCuEDiLWE2X1t_LHKVWx_f2PnuxNKtcO-F59YcLgwvPoVHCMKQOvjdB3LkQaERqKlGd0g5SXCMnNQqPHMjWJqS4qCdWBqS3NIpT6ILDlDFmQhMqMRWl6IYNnIDNB5_o7Q1FDqh3N63Qh8i7PpDvhb2Z1h9Bf1MyGcm-1p5x7ppLt_H3LCVuAIbWQdPXLuOyv9e0JxT9lZz1PZXNNXB2gbeKk9yvK8cpELCNB7q8pu1Z5J3hpzaNBOxR4qy65R3BR3l5XHxLSbkjSt3VhLi0sH6xbXxdUGH5hIMzWQ4nX9PZi-Ah4AO-YCwmUB6VVJKvqo3bz-hPQT-gqHbHx7fNFpHhZYd9rIBXAPHkFUbeyihlcV78uG-uutJeNpz1WauxuHr8TyvJqLsuTj_9Kz5cLuBZ5Ws_Bqhxk8YMVk3jSxs1oT8EPfvKxpLuJVJzTuGr0WqlwXrJcCqxf239gkQnkG4-RYjrV0r3eLQFVm-qlcNr2wO-IiRm-ZEpD-RKf_BTRG73c?type=png)

<details>
<summary>View Mermaid Code</summary>

```mermaid
graph TB
    subgraph External["External Services"]
        KYC_SVC["KYC Review Service<br/>(compliance portal)"]
        NAPIER["Napier<br/>(risk & screening)"]
        UMS["UMS<br/>(user management)"]
        INCIDENT["Incident Service"]
    end

    subgraph KafkaTopics["Kafka Topics"]
        T_REQ["kyc.request.flow<br/>KYCRequest"]
        T_NAP_OUT["napier.create.entity.kyc<br/>EntityKYBScreening"]
        T_NAP_IN["napier.create.entity.kyc.response<br/>CreateEntityKYBResponse"]
        T_DL["saga.manager.exception<br/>SagaManagerException"]
        T_JIRA["jira.ticket.details<br/>JiraTicketDetails"]
        T_NOTIF["notification<br/>JSON"]
    end

    subgraph CustomerAPI["customer-api"]
        subgraph Consumers["Kafka Consumers"]
            C_REQ["KycRequestConsumer"]
            C_NAP["CreateNapierEntity<br/>ResponseConsumer"]
        end

        subgraph Saga["Saga Engine"]
            SM["SagaManager"]
            SD["SagaDefinition<br/>10 state-to-workflow mappings"]
        end

        subgraph Workflows["Workflows"]
            W_OPS["OperationSubmitted<br/>Workflow"]
            W_OPS_APP["OperationsApproved<br/>Workflow"]
            W_SCR["ScreeningResponse<br/>Workflow"]
            W_CA["ComplianceApproved<br/>WorkFlow"]
            W_CR["ComplianceRejected<br/>Workflow"]
            W_AM["AmendTransition<br/>Workflow"]
        end

        PA["PersistCustomer<br/>ReviewStateAction"]
        DB[("CUSTOMER table<br/>CURRENT_REVIEW_STATE")]
    end

    KYC_SVC -->|publish| T_REQ
    NAPIER -->|publish| T_NAP_IN

    T_REQ -->|consume| C_REQ
    T_NAP_IN -->|consume| C_NAP

    C_REQ --> SM
    C_NAP --> SM
    SM --> SD
    SD --> W_OPS & W_OPS_APP & W_SCR & W_CA & W_CR & W_AM

    W_OPS -->|REST| INCIDENT
    W_OPS -->|produce| T_JIRA
    W_OPS -->|produce| T_NOTIF
    W_OPS_APP -->|produce| T_NAP_OUT
    W_OPS_APP -->|REST| KYC_SVC
    W_SCR -->|REST| UMS
    W_CA -->|REST| UMS
    W_CR -->|REST| UMS

    T_NAP_OUT --> NAPIER

    W_OPS & W_OPS_APP & W_SCR & W_CA & W_CR & W_AM --> PA --> DB
    SM -->|on failure| T_DL
```
</details>

---

## B. KYB State Transition Flow

![KYB State Transitions](https://mermaid.ink/img/pako:eNqNVV1v2jAU_SuWn0AFtFu3bNIeQgKBhPKxJOxhQg6JReJE_qCpiv77rnMBUjqNF8fn3HvPvXacVxLLjJKAcC53Il-TLIpTUCoXX5YayD1nhdj-yUnwVUpOtsn2F7qBJBsudgF9iHnOxIEYlAhRKLvCCBZ5uXH-FEz-pP7qnQz47I8TlsJK2Ai5M0SPcU_8JShQ-_YBqB-wfEPUPXUCadxSWWCAFbZcrEYxGBHh5XRFIjDKeG5UJjqMxYGKLBZcKUoKJrOSQ4cLBXt33T2sKU3YLaYoA8o3hIgNvT9jJFVATZBK0n2Q_P6y3dXPLr2Y5lBHxVD4r1KBa7s-DPOB4sAm5EEqnkE8GqALVaC-AH5BZCqEHGDwkqQihF9YNL4JWYzDFwqKLK8jVdCmO3uoOhMl2RxzqVK2YbnkPJdyjREtCqFCqZL5SMsD_34t0gnJ44ynfEj9C_cC3Wr9A5MtldVJWxZRpbL6Z_0rxdLDQT3g1d1J7y2eB5uyGnXTHLBcyFrfTd3RYb3aGG-Y3MIr_cPirvHGcEDqCIKhv6Ai5-Gy-c42b6dS1o0P4NdN_tPOt3RBwBqtVDVlMbL8FRPxoqLEvdBK9gG2LfmmAR7d8B4YXs2hXcT8KILh1RtYejXMX_HCaJ3hDa1t3N-vKZJvGJi-yBr6f3bAy7PEP8GYpY84Z-yX2-h5f-RYHV9sA?type=png)

<details>
<summary>View Mermaid Code</summary>

```mermaid
stateDiagram-v2
    [*] --> PENDING_ONBOARDING : POST /kyb/initiate or /kyb/redo

    state "SUBMITTED PATH" as sp {
        SUBMITTED_OPERATION
        SUBMITTED_OPS_APPROVED
        SUBMITTED_PENDING_SCREENING
        SUBMITTED_COMPLIANCE
        SUBMITTED_SCREENING_FAILURE
    }

    state "RESUBMITTED PATH" as rp {
        RESUBMITTED_OPERATION
        RESUBMITTED_OPS_APPROVED
        RESUBMITTED_PENDING_SCREENING
        RESUBMITTED_COMPLIANCE
        RESUBMITTED_SCREENING_FAILURE
    }

    PENDING_ONBOARDING --> SUBMITTED_OPERATION : First submission
    PENDING_ONBOARDING --> RESUBMITTED_OPERATION : Resubmission

    SUBMITTED_OPERATION --> SUBMITTED_OPS_APPROVED : OPS approves
    SUBMITTED_OPERATION --> AMENDED_OPERATION : OPS amends

    RESUBMITTED_OPERATION --> RESUBMITTED_OPS_APPROVED : OPS approves

    SUBMITTED_OPS_APPROVED --> SUBMITTED_PENDING_SCREENING : Sent to Napier
    RESUBMITTED_OPS_APPROVED --> RESUBMITTED_PENDING_SCREENING : Sent to Napier

    SUBMITTED_PENDING_SCREENING --> COMPLETED : Passes thresholds
    SUBMITTED_PENDING_SCREENING --> SUBMITTED_COMPLIANCE : Fails thresholds
    SUBMITTED_PENDING_SCREENING --> SUBMITTED_SCREENING_FAILURE : Napier error

    RESUBMITTED_PENDING_SCREENING --> COMPLETED : Passes thresholds
    RESUBMITTED_PENDING_SCREENING --> RESUBMITTED_COMPLIANCE : Fails thresholds
    RESUBMITTED_PENDING_SCREENING --> RESUBMITTED_SCREENING_FAILURE : Napier error

    SUBMITTED_COMPLIANCE --> COMPLETED : Compliance approves
    SUBMITTED_COMPLIANCE --> COMPLIANCE_REJECTED : Compliance rejects
    SUBMITTED_COMPLIANCE --> AMENDED_COMPLIANCE : Compliance amends

    RESUBMITTED_COMPLIANCE --> COMPLETED : Compliance approves
    RESUBMITTED_COMPLIANCE --> COMPLIANCE_REJECTED : Compliance rejects

    COMPLIANCE_REJECTED --> PENDING_ONBOARDING : Redo KYB

    COMPLETED --> [*]
```
</details>

---

## C. KYB Kafka Event Flow

![KYB Event Flow](https://mermaid.ink/img/pako:eNqVVE1v2zAM_SuCTi2QON12GXBw7Dj-iO04SttDKYmJtciSIcnpiiL_fZSdpGnRYdbB4nsk-UjRz9jnMcEOFpRteblkWRTFkGkuvkwUtP6Syuqh8s6CL4SEdSIVOrH7X8MhFcqqAilYZmWjMcaEiQ06xDJiRUjNHqQYFCIx-rrL-CPddnJF-RNqrAPKzS0K2kfP8yiJ5VKmVAVG7B3hkZQ9eMeFTLTnQUgVU5pxvS4CdAbVNuRPZJjqbewXKyJWBaaZUZ8m3sJYyOSDBWXYF6PGVBJCijCTGkKdxJGOCbzjXKGUC61TvDOcM6lYgJYiNOqPhfAeJBfU3NhH4i-6WF5lfRjIGuxd0oACkE67YzWhZwmC2UTrRD1V2y3H_FYkE5bHjCdWPxr3Wf8AGN5YyB4d3ZAdHD9ckhUTJSzjfsPOKRMqCBOSGaFwX9V1r8F8cUP2J1z9VZmWdHgqONl1vbKHWUzM5aMJuE0L7OStE8C0b-dkk3VINVtbTqFGHNY3hM8Pf_Kz8xbTcMxKm8dq1QNhGYDT8qlLAeTaEDEUlPkfZN5hNLZ6c9WxjULR4YXF3vJzjpcRaS1-Xmw43fUkiODrn_MQw2vKLBpO0ccIfIJSyR5GRwlXgPnwwHqT6hRQ3sWBGjPbKQv3QeEBuXLr_RFrAUcSqJ6d5h-0f_wEFvwEX?type=png)

<details>
<summary>View Mermaid Code</summary>

```mermaid
flowchart LR
    subgraph Producers
        KYC["KYC Review Service"]
        NAP["Napier Service"]
        CA_W["customer-api<br/>Workflows"]
    end

    subgraph Topics["Kafka Topics"]
        T1["kyc.request.flow<br/><i>KYCRequest</i>"]
        T2["napier.create.entity.kyc<br/><i>EntityKYBScreening</i>"]
        T3["napier.create.entity.kyc.response<br/><i>CreateEntityKYBResponse</i>"]
        T4["saga.manager.exception<br/><i>SagaManagerException</i>"]
        T5["jira.ticket.details<br/><i>JiraTicketDetails</i>"]
    end

    subgraph Consumers
        C1["KycRequestConsumer<br/>→ SagaManager"]
        C2["CreateNapierEntity<br/>ResponseConsumer<br/>→ SagaManager"]
        C3["External Consumers"]
    end

    KYC --> T1 --> C1
    NAP --> T3 --> C2
    CA_W --> T2 --> NAP
    CA_W --> T4 --> C3
    CA_W --> T5 --> C3
```
</details>

---

## D. KYB Happy Path Sequence

![KYB Happy Path](https://mermaid.ink/img/pako:eNqdVclu2zAQ_RWCpxSwHcdpFx98kLVYli3ZcuJDSImUOKZIgqTsOIb_vUNKduwkRRFAIDlv3mxc3kgoM0p8KpnaiXxDijDKqNJCfF0YKPeCl3L7J6X-F8UhPxZhSZX6mD3e4xn42VQIY7h9V3Kx40ruJMq7KBR8GJBPBdVQ2QBY7piMQS0hY3WxXyKNjb2yb0iqZKbUXLCNlBowsheiYMhkpwBOGMoO-Dc4TdyOqPh3b2ZcJyqT5Pj6ZqiG2iKRiowMyLz2M3Y9tXEFCKIBm5uUuCCeL39HGhdGzVYWsDtLV_XLfRiKGECjzMaSNRHZWRs2k-UhaxI_X0ypJIeJQE5HkPzWcqUNGYPQhpnKEP4NRCqQVFt8KDFUzwdvTQI_gklWLvhp4fnTLWq3KL8k1FaZnNHGnQ_cpFqPLDv3ggT7vy_e0VFZMQCnbBLpvC5zq3Tl-ZhFuxkpCCvYAyh8aIpiwnigVSkSxLdE3hEfbF5bnb3dLq36RVmgbcMN-p9BdGnxdJ-3kz-v2PPUo9Z5f6q0-95rZd2JYEdLcENyE77pBNbgJnxhZVG4hXbPhqk2D2swMdF7Sh5PBPSv4Fa9rGHVVjOrdOB5h7K6GwH21UNr3rX_d3L8gP37R8C02sNOYR28oFuWWcFpUmZyDxz-oYdrfn8qxMBzxA8e0K4u1DqnBaXfvKNaC7aVmqvNiPB0dGqwGC2O8O2Y3Jtce5dbujJQP_Bby4U9rvpXs04nMHd8_f_4DpBRQng?type=png)

<details>
<summary>View Mermaid Code</summary>

```mermaid
sequenceDiagram
    actor User
    participant API as customer-api<br/>BusinessController
    participant DB as Oracle DB<br/>CUSTOMER table
    participant KYC as KYC Review<br/>Service
    participant Kafka as Kafka
    participant SAGA as SagaManager
    participant NAPIER as Napier
    participant UMS as UMS

    rect rgb(230, 245, 255)
        Note over User, DB: Phase 1: Initiation
        User->>API: POST /kyb/initiate
        API->>DB: Save Customer(PENDING_ONBOARDING)
        API->>KYC: sendKYCRequest() [REST]
        KYC-->>API: requestId
        API->>DB: Update Customer(requestId)
        API-->>User: 200 OK
    end

    rect rgb(255, 245, 230)
        Note over KYC, SAGA: Phase 2: OPS Review
        KYC->>Kafka: KYCRequest(OPERATION)
        Kafka->>SAGA: KycRequestConsumer
        SAGA->>SAGA: OperationSubmittedWorkflow
        SAGA->>DB: Persist(SUBMITTED_OPERATION)
    end

    rect rgb(230, 255, 230)
        Note over KYC, NAPIER: Phase 3: OPS Approved + Screening
        KYC->>Kafka: KYCRequest(OPS_APPROVED)
        Kafka->>SAGA: KycRequestConsumer
        SAGA->>SAGA: OperationsApprovedWorkflow
        SAGA->>Kafka: EntityKYBScreening
        SAGA->>DB: Persist(SUBMITTED_PENDING_SCREENING)
        Kafka->>NAPIER: Screening request
        NAPIER->>Kafka: CreateEntityKYBResponse(SUCCESS)
        Kafka->>SAGA: CreateNapierEntityResponseConsumer
    end

    rect rgb(245, 230, 255)
        Note over SAGA, UMS: Phase 4: Completed
        SAGA->>SAGA: ScreeningResponseWorkflow
        SAGA->>UMS: EnableUser [REST]
        SAGA->>DB: Persist(COMPLETED)
        SAGA->>Kafka: Success notification
    end
```
</details>

---

## E. Screening Decision Tree

![Screening Decision](https://mermaid.ink/img/pako:eNp1k8tugzAQRX8FrUoUCQh9bLpAIRAI0ENI2U2IwZMiY2z5QVMl_17zEI2K2MDce2Z2bE6Y5gXBPpaM7URZkySKEqgOXj3NMN9wVor_dMT_IkNiJoo1R2kohUCGa_uruy2tmq-M1lykdUwriUmZwvqJMimV8dIsl5QkOdfo86cvKU-ZPOJpJAXswUSqgqGzjBdSkxJWQfLMVCYaxGCHFxN4V1NNRCpkBaNEUEMpL7gptcLCsrJ14jrxFqZCJu_syHr1aF1xwPeMl_DkG_ATZY23T7L3H_dOe-8kd3cxKJjkCipD_M5aCX3kqwI5gKc-QaIQq0aqUcSJ4VTD25U4ZVLzhG0I-3m1BmpknkKxUHWX-J9LHV-m-VkCRxJvCOUXJ5v5ydNgkc2vGz-LrHLGX_jWD0QRNOvGbqo7nvKQYBpQ-d6Ofib_6tXvD3TPhkb73aVg-GEI3w8kTdBH?type=png)

<details>
<summary>View Mermaid Code</summary>

```mermaid
flowchart TD
    A["Napier Screening Response Received"]
    A --> B{"Response Status?"}

    B -->|SCORE_CARD_ALREADY_EXISTS| C["COMPLIANCE"]
    B -->|FAILURE or missing data| D["SCREENING_FAILURE"]
    B -->|SUCCESS| E{"Passes risk and<br/>screening thresholds?"}

    E -->|Yes| F["COMPLETED"]
    E -->|No| G["COMPLIANCE"]

    F --- F1["Actions:<br/>Update KYC to MEDIUM<br/>Enable user in UMS<br/>Create wallet<br/>Send success notification<br/>Update entity in Napier"]
    G --- G1["Actions:<br/>Update risk details on KYC service"]
    D --- D1["Actions:<br/>Update risk details on KYC service"]

    style F fill:#d4edda,stroke:#28a745
    style G fill:#fff3cd,stroke:#ffc107
    style D fill:#f8d7da,stroke:#dc3545
    style C fill:#fff3cd,stroke:#ffc107
```
</details>

---

## F. Saga Workflow Engine Internals

![Saga Engine](https://mermaid.ink/img/pako:eNqNVF1v2jAU_SuRn4YEtFu3qdIeAgkEEsrH0rCHCTkklokT-YOlqvrvuwnhI-262Xx87jn33mudr0msckYDIri8iGxD8ihJqdZCfVkYJA-ClXJ3kbPgUSlBDjw_kfKIpCNAd5xJLCJ90S5Sn0lxBaD2Tq0fkLXe0LHZO24qNsKfkiVUAp6lUgolYR1SnMQqMQN3EAlQr0T8Fx4vubiioqJ6Cp4YFGVCRWNjC5YJuQHD8qSAjEuSSYGrGxMFLIv4z5TI01TOmaUv2BLmkzIcaGBBhGbCtdJg5A7XuGzsBDcbM3IlKYbGJm11eBjnLHVRxSPIKCjH78TzF0iWMbnPiGx_4kfWkq5YMkWdWRMJ18sLpOjW6rJx5iQKWrVfz2nGqEBf9CaKJgVXV7ueOmJi_k3PJa78TWsKLxJdz5uZJMaPfEnNi95L3EQh91-2aNE2K8p-u4qqDQMD6l67sJ6MrT3vHh4u55W9lzf2iF1hPBLqtN-MxDM2l8pFJl7NZFiAA31n2tS-NjKCsNVlQpnnHOKn3vTF2Bnm8sWB6Hd_3jVLvp9LUY91kvjv_B4K4-9k-v_w47fHn_D7D8BA?type=png)

<details>
<summary>View Mermaid Code</summary>

```mermaid
flowchart TD
    A["Kafka Consumer receives event"]
    A --> B["Resolve ReviewStateEnum from event"]
    B --> C["Create KycContext with state + data"]
    C --> D["SagaManager.execute()"]
    D --> E["SagaDefinition.getWorkflow(state)"]

    E -->|null| F["Log + silently discard"]
    E -->|found| G["workflow.getTargetState()"]
    G --> H["kycContext.nextState(target)"]
    H --> I["workflow.execute()"]
    I --> J["Run action chain sequentially"]
    J --> K["PersistCustomerReviewStateAction"]
    K --> L["DB: Customer.currentReviewState = newState"]

    J -->|exception| M["SagaManager.rollback()"]
    M --> N["getFailureTargetState()"]
    N --> O["Run rollback actions"]
    O --> P["Persist failure state to DB"]
    P --> Q["Publish to saga.manager.exception topic"]

    style F fill:#f8d7da,stroke:#dc3545
    style L fill:#d4edda,stroke:#28a745
    style Q fill:#fff3cd,stroke:#ffc107
```
</details>

---

## G. Proposed Future: Legacy + eKYC Dual Flow

![Proposed Dual Flow](https://mermaid.ink/img/pako:eNqVVNtu2zAM_RVBTwuQON0t2OBH27GdxLYcJ-1DKImxNcuSIcnpiiD_PtmXXNBhegB5eHh4SNE3xFMZIxcxxjY82xFREk8AkFJ-WSjI94yVYvEnQe4XI0WVJJI2cXhLJVciVJkOxJOAiIBlgkZ0R4QzJ2L_6P8uQk3JQ-WYxOlKCB9IodUOtBqwbKX1IcqXvJC0S7f_2oIu26gVVyqVvDQtWGioCyTbrpRMEd0RmcJKyNyoGPQ0OuRhQjM07KUMlYbfWq-NVcLYDW5tgjFbM5mnKkQvHpNS-eT8E4S2SZ3hMt8vV9TIlNM3Fk_-NpGpMFSn2JYpzVcjS5_QDFBghyJOd4h_V0f2Osp_1xJ7oHl1J8pu5YqoYGb3dNxI-kMr7d3twT3S-cLQ-QlQOVGtOCxA-MSsIWoQQtqrGo66v-KbGxNdlA71r3t7UdjJj0iCgD_tVu3mX7rxh8Y29jAHHG78M8JI4k-9cLsQN3q7UuoFEvx5N1OOuK3t3bbsUvSu8C2l1qOq-Bqj2Whr3qNl8OHtjfhNOz74KSmqPLZ0Ns3R4PNn_8BmIm0aQ?type=png)

<details>
<summary>View Mermaid Code</summary>

```mermaid
flowchart TB
    ENTRY["POST /kyb/initiate"]
    ENTRY --> ROUTER{"Flow Router<br/>isUsingNewOnboarding?"}

    subgraph LEGACY["LEGACY KYB Flow"]
        direction TB
        L1["PENDING_ONBOARDING"]
        L2["SUBMITTED_OPERATION"]
        L3["SUBMITTED_OPS_APPROVED"]
        L4["SUBMITTED_PENDING_SCREENING"]
        L5["COMPLETED"]
        L6["SUBMITTED_COMPLIANCE"]
        L7["COMPLIANCE_REJECTED"]
        L1 --> L2 --> L3 --> L4
        L4 -->|pass| L5
        L4 -->|fail| L6
        L6 -->|approve| L5
        L6 -->|reject| L7
    end

    subgraph EKYC["eKYC Flow — Proposed"]
        direction TB
        E1["EKYC_PENDING"]
        E2["EKYC_IDENTITY_VERIFIED"]
        E3["EKYC_PENDING_SCREENING"]
        E4["EKYC_COMPLETED"]
        E5["EKYC_MANUAL_REVIEW"]
        E6["EKYC_REJECTED"]
        E1 --> E2 --> E3
        E3 -->|pass| E4
        E3 -->|fail| E5
        E5 -->|approve| E4
        E5 -->|reject| E6
    end

    ROUTER -->|Legacy| L1
    ROUTER -->|eKYC| E1

    subgraph SHARED["Shared Infrastructure"]
        SM["SagaManager + SagaDefinition"]
        LIB["kyc-review-state lib"]
        KAFKA["Kafka Topics"]
        DATABASE[("CUSTOMER table")]
    end

    L5 & L7 --> SM
    E4 & E6 --> SM
    SM --> LIB
    SM --> KAFKA
    SM --> DATABASE

    style L5 fill:#d4edda,stroke:#28a745
    style E4 fill:#d4edda,stroke:#28a745
    style L7 fill:#f8d7da,stroke:#dc3545
    style E6 fill:#f8d7da,stroke:#dc3545
```
</details>
