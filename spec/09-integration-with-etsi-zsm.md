### 9. Integration with ETSI Zero-touch network and Service Management (ZSM)

This section describes the integration of the ACE-NET framework with the ETSI ZSM reference architecture. The integration positions ACE-NET as a specialized `management service` that provides automated compliance, validation, and security assurance for network agents and functions within a zero-touch operational environment.

#### 9.1. Architectural Mapping

The ETSI ZSM framework is a service-based, modular architecture designed for closed-loop automation at scale. It comprises Management Domains (MDs) and an End-to-End (E2E) Service Management Domain connected by an Integration Fabric. ACE-NET maps into this architecture as a dedicated, cross-domain `management service` for agent compliance.

*   **ACE-NET Core as a ZSM Management Service**: The ACE-NET Core Engine is exposed as a `management service` within the ZSM ecosystem. Other ZSM functions, like an orchestrator, can consume ACE-NET's services via its published APIs (see Section 6) to orchestrate tests and manage certificates.
*   **TEA and Management Domains**: The ACE-NET TEA acts as a bridge to the ZSM Management Domains (e.g., RAN, Core, Transport). The TEA interacts with domain managers to provision test environments and collect data.
*   **Mapping to ZSM Functional Blocks**:
    *   **Orchestration Services**: ACE-NET's test orchestration complements ZSM's service orchestration. While ZSM orchestrates the service lifecycle, ACE-NET orchestrates the compliance validation lifecycle as a critical sub-process.
    *   **Analytics and Data Services**: The ACE-NET reporting and audit data (see Section 5.3) aligns with ZSM's `Data Services`. It provides compliance-specific insights that can feed into broader ZSM analytics functions.
    *   **Policy Management**: The ACE-NET Policy Engine is a specialized policy point for compliance. It consumes intent-based policies from the ZSM framework and translates them into testable assertions using the `ace-policy.yang` model.

#### 9.2. Operational Flow in the ZSM Service Lifecycle

ACE-NET integrates directly into the ZSM service lifecycle management (LCM) process.

1.  **Onboarding and Pre-Deployment**: When a new agent is introduced into the ZSM service catalog, the ZSM orchestrator triggers the ACE-NET `management service` to initiate a compliance test run by submitting the agent's profile via the API.
2.  **Certification within Automation Loops**: Agent certification becomes a key gate within ZSM's automation loops.
    *   **Open Loop**: An operator can mandate that all new agent versions must pass ACE-NET certification before being enabled in the service catalog.
    *   **Closed Loop**: A ZSM orchestrator can programmatically call the ACE-NET API to verify a valid compliance certificate before deploying or upgrading an agent, proceeding only upon success.
3.  **Continuous Monitoring**: ZSM service assurance tools can use ACE-NET for continuous validation. If a ZSM monitor detects an anomaly, it can trigger a targeted ACE-NET test run to re-validate the agent's compliance posture.

#### 9.3. Interaction with ZSM Closed-Loop Automation

ZSM's closed-loop automation relies on a "collect-analyze-decide-act" cycle. ACE-NET provides critical capabilities for this cycle in the context of compliance.

*   **Collect**: The ACE-NET TEA collects fine-grained data (logs, metrics, state) from the AUT during a test run.
*   **Analyze**: The ACE-NET Core Engine analyzes this data against policies defined in `ace-policy.yang`, producing a structured compliance report.
*   **Decide**: A ZSM `management function` consumes the ACE-NET report. A compliance failure could lead to a decision to quarantine the agent or roll back an update.
*   **Act**: The ZSM orchestrator executes the decision, for example, by removing the agent from service or blocking its network access.

#### 9.4. Policy Enforcement with `ace-policy.yang`

ETSI ZSM promotes intent-based interfaces that express *what* should be achieved, while policies define *how*. The `ace-policy.yang` model provides a structured format to define the "how" for compliance. An SLO defined as a ZSM intent (e.g., "Ensure all UPF agents are hardened against specific vulnerabilities") can be implemented using `ace-policy.yang`. The policy would define the specific `test-stimulus`, `monitoring-probes`, and `assertions` required to validate that intent. The ZSM framework can manage the high-level intent, while ACE-NET executes the detailed technical validation and reports the outcome.
