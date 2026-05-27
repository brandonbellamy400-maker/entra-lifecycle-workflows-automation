# 🆔 Automated Enterprise Identity Governance & API Orchestration Lab

## 🏢 Business Scenario: The Offboarding Risk Window
* **Company Problem:** Global enterprise logistics provider **BrandonTech** identified a critical security vulnerability during employee termination cycles. When an employee was offboarded, human latency meant accounts remained active for hours—or days—leaving a wide exposure window for data exfiltration, unauthorized access, and insider threats. Furthermore, downstream operational tasks (such as routing mailbox contents or identifying data ownership) stalled because IT lacked a mechanism to programmatically map a departed employee's structural reporting line at the exact moment of termination.
* **The Solution:** As an Identity and Access Management (IAM) Engineer, I engineered and deployed an event-driven **Identity Governance and Administration (IGA)** automation pipeline. By leveraging **Microsoft Entra ID Lifecycle Workflows**, I automated immediate account revocation. To bridge directory events with business logic, I deployed an enterprise-tier serverless **Azure Standard Logic App** to parse real-time webhook payloads and programmatically query the **Microsoft Graph API** backend, dynamically extracting and exposing management data exactly at the time of termination.

## 🛠️ Skills and Technologies Demonstrated
* **Identity Governance & Administration (IGA):** Entra ID Lifecycle Workflows & Custom Extensions
* **Cloud Architecture & Integration:** Azure Standard Logic Apps, Webhook Handshakes, & API Routing
* **API Engineering:** Microsoft Graph API, JSON Payload Parsing, & Custom Expression Formulation
* **Enterprise Troubleshooting:** Log Diagnostics, REST API Error Analysis, & Pipeline Refactoring

---

## 🚀 Step-by-Step Lab Walkthrough

### Phase 1: Directory Governance & Lifecycle Engineering
To completely mitigate the termination exposure window, I established a native **Entra ID Lifecycle Workflow** leveraging the *Real-time employee termination* template. This configuration ensures that the second an HR termination event is logged, a programmatic sequence executes: the identity is instantly disabled globally, and all active authentication refresh tokens are forcefully revoked—truncating any active sessions across mobile devices, web browsers, and corporate endpoints.

To extend this workflow beyond standard directory boundaries, I registered an Entra ID **Custom Workflow Extension**, transforming the identity event into an outbound webhook capable of triggering microservices across the corporate infrastructure.

### Phase 2: Serverless Pipeline & Graph API Architecture
I provisioned an enterprise-tier, stateful **Azure Standard Logic App** (`logic-entra-lifecycle-standard`) to act as the web-receiver for Entra's lifecycle events. Inside the **Logic App Designer**, I built a 3-step structured API processing graph:
1. **HTTP Trigger Receipt:** Intercepts the raw JSON payload dispatched from the Entra ID directory.
2. **Graph API Traversal (`Get manager (V2)`):** Parses the incoming `Subject.Id` context string and queries the live corporate organizational structure via Microsoft Graph to extract the departed user's direct reporting line.
3. **Structured HTTP Response:** Compiles the management metadata and returns a clean `200 OK` status back to the directory backend with the manager's explicit email mapped inside the response body using the native Azure expression: `@{body('Get_manager_(V2)')?['mail']}`.

![Logic App Designer Architecture](Screenshot%202026-05-25%20191334.png)

---

### Phase 3: Live Verification & Webhook Execution
To validate the reliability of the cross-cloud handshake, I executed an on-demand deployment against a test identity target pool. The Azure Portal **Trigger History** log explicitly confirms the structural validity of the pipeline, recording a successful HTTP handshake execution:

![Azure Logic App Trigger Success](Screenshot%202026-05-25%20192356.png)

---

### Phase 4: Production Logging & Incident Analysis 
The final production audit dashboard within the **Microsoft Entra Admin Center** displays the complete execution timeline. Rather than a synthetic "flawless" deployment, this log preserves the true engineering lifecycle, documenting initial failures, root-cause identification, and ultimate remediation:

![Entra ID Lifecycle Workflow History](Screenshot%202026-05-25%20193229.png)

#### 🔍 Root-Cause Analysis & Engineering Remediation
As a Cloud Engineer, observing and debugging failures is where architectural resilience is built. The ledger above tells an authentic technical story:

1. **The Initial Incident (User: `Chris Green` at 6:51 PM & 6:54 PM):**
   * **Symptom:** Entra ID marked the workflow execution as **Failed / Completed with errors**.
   * **Root-Cause Triage:** I isolated the bottleneck to the outbound `Trigger-Manager-Offboarding-Review` step. The original design called an Office 365 Outlook mail-relay action. However, because this sandbox environment utilizes a free Microsoft Developer Tenant, the administrator account lacked a paid Exchange Online subscription license, throwing a native `MailboxNotEnabledForRESTAPI` authentication error.
2. **The Architectural Fix:**
   * I refactored the Logic App, deleting the broken Outlook dependency entirely. I replaced it with a streamlined, zero-license **HTTP Response action block**, capturing the Graph API's extraction metrics and outputting them natively back to the directory stream.
3. **The Production Victory (User: `Bob Jones` at 6:52 PM):**
   * **Result:** Status shifted to **Completed Successfully**.
   * With the code modifications saved, I ran a second on-demand deployment against `Bob Jones`. The pipeline executed flawlessly, instantly stripping the user's active tokens and feeding the management structure metrics smoothly back to the tenant records.

---

 ## 🎯 Engineering Takeaways & Architectural Insights
* **Event-Driven Identity Architecture:** This project demonstrates the execution of event-driven directory governance rather than relying on stale, delayed scheduled scripts. By bridging native identity providers with serverless cloud infrastructure via webhooks, account revocations scale seamlessly across the enterprise.
* **Declarative Data Engineering:** Building this pipeline required strict structural manipulation of payload data streams. Navigating JSON schemas, formulating custom Microsoft Graph API tokens, and dynamically mapping operational arrays ensure zero manual human overhead during business workflow transitions.
* **Resilient Infrastructure Debugging:** The engineering lifecycle of this deployment emphasizes the value of rigorous log diagnostics. Confronting local environment sandboxing blocks (such as the Exchange native `MailboxNotEnabledForRESTAPI` restriction) provided a direct path to refactor dependencies, resulting in a leaner, zero-licensed native HTTP Response framework.
