# Proposed Dual Flow: eKYC and Legacy KYB Integration

---

## 1. Executive Summary

This document proposes a unified **Dual-Flow Onboarding Architecture** that integrates a modern **eKYC (automated individual identity verification)** flow with our **Legacy KYB (Know Your Business)** flow.

Under this architecture:
1. **eKYC** acts as the front-end automated gatekeeper to verify the identity of the corporate/merchant representative (individual). It utilizes automated OTP verification, dual-sided Emirates ID scanning with OCR, and FaceTec 3D Liveness and Facial Matching.
2. **Success/Fallback Handling** splits verified users into an automated happy-path, while routing questionable/failed cases to manual compliance queues.
3. **Transition to KYB**: Once individual identity (eKYC) is verified, the flow smoothly hands over to the **Legacy KYB Flow** to verify business credentials (Trade License, UBOs, Shareholders, Napier corporate screening, and operations/compliance approvals).

---

## 2. Integrated Onboarding Pipeline

```
[ Individual eKYC Phase ] 
      │ (OTP → Emirates ID OCR → FaceTec 3D Liveness Check)
      ▼
[ Automated Verification Decision ]
      ├─── Pass ───► [ KYB Business Phase ] (Trade License, Shareholders, Napier Screen)
      └─── Fail ───► [ Manual Review Phase ] (Compliance Queue/Portal Override)
```

---

## 3. Detailed Step-by-Step Flow

### Phase 1: Individual eKYC Verification
1. **Initial Registration**: The corporate/merchant representative signs up on the portal with an Email and Mobile number.
2. **OTP Verification**: The system sends and verifies OTP on both channels (SMS and Email) to guarantee communication ownership.
3. **Emirates ID (EID) Upload**:
   - The user scans the front and back of their Emirates ID.
   - The system performs **OCR extraction** to fetch demographic details (Name, EID Number, Date of Birth, Expiry, Nationality) and the embedded photo.
   - External validation (ICA/ICP or UAE Pass) verifies card authenticity and validity.
4. **FaceTec 3D Liveness Check & Matching**:
   - The user performs a 3D selfie scan using the FaceTec SDK.
   - The FaceTec server performs **3D Liveness Detection** to prevent spoofing (e.g., photos, video playbacks, masks).
   - FaceTec performs **Facial Matching (1:1)** between the 3D selfie map and the photo extracted from the scanned Emirates ID.

### Phase 2: Automated Verification Guard
The outcome of Phase 1 determines the next step:
- **eKYC SUCCESS**: If Liveness is verified and Face Match score exceeds the configured threshold:
  - Create the `IamUser` and `IamEntity` profiles.
  - Set KYC status to `MEDIUM`.
  - Automatically transition directly to the **KYB Business Details** collection.
- **eKYC FAILURE**: If liveness fails, or face match is low:
  - Route the signup request to the **Manual Compliance Queue**.
  - Review Status is updated to `PENDING_FACETECH` or `PENDING_ADDITIONAL_INFO`.
  - An operations or compliance officer can manually approve, reject, or request amendment via the portal.

### Phase 3: Transition to Legacy KYB Flow (Business Phase)
Upon successful eKYC verification, the representative registers the business:
1. **Business Registration**: The representative inputs business details (Trade License number, Business Type, Corporate Name, and uploads company documents).
2. **KYB Initiation**: The system executes `InitiateKybCommand` which:
   - Links the verified representative to the business entity.
   - Creates a `Customer` record in the database with `currentReviewState = PENDING_ONBOARDING`.
   - Sends a request to the KYC Review service, receiving a `requestId`.
3. **Operations Review (`SUBMITTED_OPERATION`)**:
   - The workflow triggers `OperationSubmittedWorkflow`.
   - An incident and Jira ticket are created, and a notification is sent.
   - Operations reviewers examine the trade license and business documents in the compliance portal.
4. **Operations Approved (`SUBMITTED_OPERATIONS_APPROVED`)**:
   - Once operations approves, the `OperationsApprovedWorkflow` triggers.
   - The system sends the company and shareholder details to **Napier** for AML/Screening.
   - State updates to `SUBMITTED_PENDING_SCREENING`.
5. **Napier Screening Response**:
   - **Pass**: If screening passes, state moves to `COMPLETED`. The user is enabled in UMS, wallet is created, and success notifications are sent.
   - **Fail/Compliance**: If screening fails or requires manual compliance review, the state transitions to `SUBMITTED_COMPLIANCE`. Compliance officers must manually approve or reject, which subsequently leads to `COMPLETED` or `COMPLIANCE_REJECTED` states.

---

## 4. Visual Diagrams

### Diagram A: End-to-End Proposed Dual Flow (eKYC to Legacy KYB)

![End-to-End Proposed Dual Flow](https://mermaid.ink/img/pako:eNqVVdtu4jAQ_RXLK7VVkS7S3R6kh-WCEpYsKCyLSTePFXLAKVaxDf77OAnNshS6uY_JnDnzmLF5xlS6BDuYUrXh6ZqkURSDVJvky0pBeSIsFds_OfW-MKFSJKmokmBJlUR3pvz1Z0pC_DpsXG8_vT9R_70W7RIsA-FmRFVv7f17bWvXF6pG7r6N3r-0vF3MepreKshqL2XlyrB9Yy0OaEOInqE7-Iu5H-l9bH5gsh_TAnPpxWAsXjD_8X_4_D867wK2rOAhF_uHttA12-v4b-FpM_AbeLof-Aae7PuhgSe-b70hOQbyC_tAbL90qKDoF6H9Q7S_W49p42j7u-bAn_GAt-OIs5DqIDr6EorD2F7O-X6P_C17X_tXf2Dsc73r8X8r8J3x-L8XwZ0f7SbywE6y-Gv7PjOxlvW6iR8FhT7yF8bXGep-9f6_U3PMyL_wX8Y9D3wE_Z_0_q_Wv5oDPh_w9T5C724X-EaorM7asogqk9U_K18plv7u_gNf3R71_uF5YFf_TIDmveXAnPebp-3v2s-Z7tFhZzTqPveqLInxMyL_G3-G0fXUuXm69h_rZg_GofZpW8D7R_i7Bsc0IuPTeK79nOlPveS4x1Y7mZ7m9N89fndXF6T_iL_U9f-5f_TAt8f70N0m0nscS3T_HPhSg0M-mX4_N6G9T827m-bC8p89_rDndH_43V3O-yM-mX6_uPscf7qg26_jP8R_R_Q_8df_D?type=png)

<details>
<summary>View Mermaid Code</summary>

```mermaid
flowchart TD
    subgraph eKYC_Phase["Phase 1: Individual eKYC"]
        Start(["User Registration"]) --> EmailMobile["Enter Email & Mobile"]
        EmailMobile --> VerifyOTP["Verify Dual OTP (SMS & Email)"]
        VerifyOTP --> ScanEID["Scan Emirates ID (Front & Back)"]
        ScanEID --> ExtractOCR["Extract OCR Details & Photo"]
        ExtractOCR --> FaceTecSDK["FaceTec 3D Liveness Selfie Scan"]
        FaceTecSDK --> FaceTecMatch{"FaceTec Match Score<br/>& Liveness Success?"}
    end

    subgraph Fallback_Phase["Manual Fallback Phase"]
        FaceTecMatch -->|No / Failed| ReviewManual["Update reviewStatus to<br/>'PENDING_FACETECH'"]
        ReviewManual --> ManualDocs["Upload Documents Manually"]
        ManualDocs --> ComplianceReview["Compliance Manual Queue"]
        ComplianceReview --> ManualApprove{"Compliance Manual<br/>Approved?"}
    end

    subgraph KYB_Phase["Phase 2: Legacy KYB Flow"]
        FaceTecMatch -->|Yes / Passed| RegisterBiz["Register Business Details<br/>(Trade License & Docs)"]
        ManualApprove -->|Yes| RegisterBiz
        RegisterBiz --> InitiateKYB["Execute InitiateKybCommand<br/>DB: Customer(PENDING_ONBOARDING)"]
        InitiateKYB --> SubmittedOps["Saga: OperationSubmittedWorkflow<br/>DB: Customer(SUBMITTED_OPERATION)"]
        SubmittedOps --> OpsReview{"Operations Team Approved<br/>Trade License & Docs?"}
        
        OpsReview -->|Yes| ApprovedOps["Saga: OperationsApprovedWorkflow<br/>DB: Customer(SUBMITTED_OPS_APPROVED)"]
        OpsReview -->|Amend| AmendOps["Saga: AmendTransitionWorkflow<br/>DB: Customer(AMENDED_OPERATION)"]
        
        ApprovedOps --> SendScreening["Publish EntityKYBScreening to Kafka<br/>DB: Customer(SUBMITTED_PENDING_SCREENING)"]
        SendScreening --> NapierScreen["Napier Corp & UBO Screening"]
        
        NapierScreen --> NapierResult{"Screening Result?"}
        
        NapierResult -->|Pass| OnboardingSuccess["Saga: ScreeningResponseWorkflow<br/>DB: Customer(COMPLETED)"]
        NapierResult -->|Fail / Compliance| SubmittedCompliance["Saga: ScreeningResponseWorkflow<br/>DB: Customer(SUBMITTED_COMPLIANCE)"]
        
        SubmittedCompliance --> ComplianceDecision{"Compliance Officer Approved?"}
        ComplianceDecision -->|Yes| OnboardingSuccess
        ComplianceDecision -->|No| RejectKYB["Saga: ComplianceRejectedWorkflow<br/>DB: Customer(COMPLIANCE_REJECTED)"]
    end

    OnboardingSuccess --> EnableUMS["Enable User in UMS & Create Wallet"]
    EnableUMS --> EndOnboarding(["Onboarding Completed Successfully ✓"])
    RejectKYB --> EndOnboarding
    ManualApprove -->|No| RejectIndividual["Reject Representative Signup"]
    RejectIndividual --> EndOnboarding
```
</details>

---

### Diagram B: Automated Screening Decision Flow Chart

This tree details the automated screening evaluation block from the end of the eKYC phase through the Napier corporate lookup.

![Screening Decision](https://mermaid.ink/img/pako:eNqVk91ugzAMhV8l8m6pkEDpY4sCCQQIdBBS6mRCDpYmGbFlrCIp0qdX_pYmYVvXTe7N-dk-OnPAtM4IdpEkbF-WW-IQRRDUO9c8H7DYcJrz-3LAnyFDYiryFc9Skyv7m_uru62Nms6NFlzEcUQLiSmRwfKeMinp_ZIFCjKX6DOH_m-R8gT6I55GXMg_DAnY_Y02pDoREfXQvKMs6vFqCO-mbygT9ZAL9M1EAt-rV0wJxSp-v_068Qomf8Yv78Zqg8sM6t-rVkID9epA_Lskbndf0D1_3LvlPQ95g9Yxv2UthZp6ViAncNMfSAzW9ED9e6_WQCXN64D_vFsDLTInUBZk_SX8Z6nju8w8S-BE4hmg7NJkM8-FBi1tXp_8TKL6_f_K97zR68btoR6G8f1A9gZ9p_L-?type=png)

<details>
<summary>View Mermaid Code</summary>

```mermaid
flowchart TD
    A["Receive Napier Screening Response"] --> B{"Response Status?"}

    B -->|SCORE_CARD_ALREADY_EXISTS_FAILURE| C["Transition to SUBMITTED_COMPLIANCE"]
    B -->|FAILURE / Missing Data| D["Transition to SUBMITTED_SCREENING_FAILURE"]
    B -->|SUCCESS| E{"Entity passes Risk &<br/>Screening Thresholds?"}

    E -->|Yes| F["Transition to COMPLETED<br/><br/>Automated Actions:<br/>1. Update KYC status to MEDIUM<br/>2. Enable User in UMS<br/>3. Execute Create Wallet Command<br/>4. Publish Successful Onboarding Notification<br/>5. Send update confirmation to Napier"]

    E -->|No| G["Transition to SUBMITTED_COMPLIANCE<br/><br/>Manual Actions:<br/>1. Update Risk Details on KYC Service<br/>2. Present to Compliance Queue for Override"]

    C --> G
    D --> H["Manual Operations Review required"]

    style F fill:#d4edda,stroke:#28a745
    style G fill:#fff3cd,stroke:#ffc107
    style D fill:#f8d7da,stroke:#dc3545
```
</details>

---

### Diagram C: Interaction Sequence Diagram (eKYC to Legacy KYB transition)

This sequence diagrams the transition where the successful automated eKYC flow resolves into the legacy KYB command setup.

![Transition Sequence](https://mermaid.ink/img/pako:eNqdVV1v2jAU_SuWn0AFtFu3adIeQgKBhPKxJOxhQg6JReJE_qCpiv77rnMBUmZra_HxeZ1777XjZyIxLwgOCediy5INGRKnIDTKnxcK8ANnnS-fV8b7vDCSZ1X8LAmXpUonNnt7_5gI0eU1v6Y8p33WeWssj1hRe8XvL0F6v07e5y6ZisG_C1r_9N_bV9tb3nU69EfeL68H6B7x96r677Vov6P3r22vD8shKzmWp_a8_e9qO7A80C9oWv7FPK70PhZfMdkL-v6G2VfW_b_06D8at_ZshfFIsG_b9O6m0u7A8UAvUHH5L_O48vdYfMNkz-v7S9byYfX_p8_R_iN6t3N-r7Fp5z8f_1ndP9M9O_6z_He0v_vYp_8D69t-gA?type=png)

<details>
<summary>View Mermaid Code</summary>

```mermaid
sequenceDiagram
    actor User as Corporate Representative
    participant UI as Portal / Web App
    participant CAPI as customer-api<br/>(SignUpCommand)
    participant FT as FaceTec SDK & Server
    participant DB as Oracle DB
    participant KYBS as Saga Manager<br/>(InitiateKybCommand)

    rect rgb(230, 245, 255)
        Note over User, FT: Phase 1: eKYC Verification
        User->>UI: Enter Email, Mobile & Verify OTP
        User->>UI: Scan front/back of Emirates ID
        UI->>CAPI: Upload scanned images & OCR data
        User->>UI: Take 3D Liveness Selfie
        UI->>FT: Send Liveness Selfie & EID Photo for Matching
        FT-->>UI: Liveness: Success, Match Score: 98%
    end

    rect rgb(255, 245, 230)
        Note over UI, DB: Phase 2: User Account Creation (eKYC Success)
        UI->>CAPI: Trigger Individual Signup
        CAPI->>DB: Insert IamUser, IamEntity (kycStatus=MEDIUM)
        CAPI-->>UI: 200 OK (User Profiles Created)
    end

    rect rgb(230, 255, 230)
        Note over User, KYBS: Phase 3: Transition to Legacy KYB
        User->>UI: Enter Company Name, Trade License & Upload Docs
        UI->>CAPI: POST /business/kyb/initiate
        CAPI->>KYBS: Execute InitiateKybCommand
        KYBS->>DB: Insert CUSTOMER (ReviewState=PENDING_ONBOARDING)
        KYBS-->>UI: 200 OK (KYB Process Initialized)
        Note over KYBS: Handover to Legacy KYB Saga engine...
    end
```
</details>

---

## 5. Architectural Alignment & Recommendations

### 5.1 Clear Separation of Context
The `customer-api` currently suffers from conflating **User Registration** and **KYB Customer Registration**. Integrating the eKYC dual flow allows a clean separation of roles:
- **`IamUser`/`IamEntity`**: Store the individual representative's identity details and their automated eKYC liveness verification status (`IamFaceTecEnrollment` / `IamFaceTecIdScan` entities).
- **`Customer`**: Stores the merchant or corporate business entity details, mapped to `currentReviewState` of type `ReviewStateEnum`.

### 5.2 Clean Handover Mechanism
1. **eKYC Stage**: Complete representative's registration using the existing `IndividualEntitySignupCommand`. Perform Emirates ID extraction and FaceTec verification synchronously.
2. **Representative Status**: If eKYC passes, save `IamUser.reviewStatus = CUSTOMER` and `kycStatus = MEDIUM`.
3. **Business Registration Trigger**: Once representative is verified (`CUSTOMER` status), enable the trade license collection UI.
4. **KYB Stage**: On submission, execute `InitiateKybCommand`. This writes to the `CUSTOMER` table with `PENDING_ONBOARDING` and registers a request on the KYC Review Service. The background state transitions (`kyc.request.flow` Kafka topic events) now manage operations reviews and corporate Napier screenings.
