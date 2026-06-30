### 2. Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 [RFC2119] [RFC8174] when, and only when, they appear in all capitals.

**Agent:** Any autonomous or semi-autonomous software component that observes network state, makes decisions, and performs actions within a telecom environment. Examples include AI/ML-driven AIOps engines, network slice management functions, RAN optimization xApps/rApps, and OSS/BSS automation controllers.

**Agent Under Test (AUT):** An Agent that is currently undergoing compliance certification by ACE-NET.

**Agent Compliance Engine for Networking (ACE-NET):** The logical system that orchestrates the multi-layer compliance certification of telecom Agents. ACE-NET is a telecom-domain specialization of the open-source ACE framework [REF-ACE].

**ACE Certification Pipeline:** The four-phase pipeline inherited from the base ACE framework: Phase 0 (fault bucketing), Phase 1 (metrics extraction), Phase 2 (statistical aggregation), Phase 3 (certification report generation). See Section 4.

**Agent Profile:** A machine-readable document describing an Agent's identity, capabilities, declared interfaces, autonomy level claim, A2A-T endpoint, and target HVS scenarios. See Section 5.1.

**Compliance Certificate:** A signed JWT issued by ACE-NET upon completion of all applicable certification layers. Contains both the `autonomy_level_claimed` by the agent and the `autonomy_level_achieved` as computed by ACE-NET. See Section 4.5.

**Telecom Environment Adapter (TEA):** An ACE-NET component providing a uniform command surface over heterogeneous telecom interfaces. Comprises six plugin types: 3GPP NBI, O-RAN O1/A1/E2, NETCONF/YANG, TMF OpenAPI, A2A-T, and Chaos Injector. See Section 3.2.

**Domain Tool Interface:** The standardized API layer through which a telecom agent observes network state and effects changes. In the telecom domain, TMF Open APIs serve this role — analogous to the Model Context Protocol (MCP) in general-purpose AI agent frameworks. TMF API call correctness is a first-class certification dimension in ACE-NET.

**TMF Open APIs:** The suite of TMForum-standardized REST APIs used as the domain tool interface for telecom agents. Relevant APIs in ACE-NET: TMF628 (Performance Mgmt), TMF639 (Resource Inventory), TMF640 (Activation & Config), TMF641 (Service Order), TMF654 (Trouble Ticket), TMF688 (Event Mgmt), TMF724 (AI/ML Model Mgmt).

**A2A-T (Agent-to-Agent Protocol for Telecom):** The inter-agent communication protocol defined in TMForum IG1453. Enables agents to publish capabilities via Agent Cards, discover peer agents via a Registry Center, delegate sub-tasks, and coordinate multi-domain workflows via the IG1453A Prompt Meta-Model.

**Agent Card:** A JSON document published by an A2A-T-compliant agent at `/.well-known/agent.json`, declaring the agent's identity, skills, authentication schemes, and IG1453A Prompt Meta-Model capability. Validated in Layer 2 Phase A.

**Agent Fabric:** The A2A-T runtime environment defined in TMForum Catalyst C26.0.910, comprising the Registry Center and Orchestration Center. Provides the infrastructure for multi-agent coordination in IG1401 L4 networks.

**Registry Center:** The IG1453 component that manages Agent Card publication, authentication, and skill-based agent discovery. Agents resolve peer endpoints through the Registry Center rather than using hardcoded addresses.

**Orchestration Center:** The IG1453 component that coordinates multi-agent workflows using the IG1453A Prompt Meta-Model. Used to dispatch HVS scenarios in Layer 3 certification.

**IG1453A Prompt Meta-Model:** The structured prompt schema defined in IG1453A that governs how tasks are expressed, dispatched, and responded to within the Agent Fabric. Compliance with this schema is validated in Layer 2 Phase D.

**High-Value Scenario (HVS):** A TMForum-defined end-to-end telecom scenario from the IG1401 Level 4 Blueprint that exercises autonomous agent capabilities across one or more domains. ACE-NET defines five HVS identifiers: HVS-1 (Fault Management), HVS-2 (Zero-Touch Slicing), HVS-3 (Self-Optimizing RAN), HVS-4 (Proactive SLA Assurance), HVS-5 (Complaint Handling). See Section 8.

**Autonomy Level:** The TMForum IG1218 classification of network automation capability: L0 (Manual), L1 (Assisted), L2 (Partial Self-X), L3 (Conditional Self-X), L4 (High Self-X). ACE-NET certification tiers map directly to these levels.

**Certification Tier:** The ACE-NET tier (0–4) corresponding to an IG1218 Autonomy Level. Each tier defines the applicable certification layers, fault scope, and HVS requirements. See Section 3.4.

**Three-Layer Certification Model:** The three cumulative evidence layers in ACE-NET: Layer 1 (Behavioral Certification), Layer 2 (A2A-T Protocol Certification), Layer 3 (HVS Scenario Certification). Together they feed the ACE Phase 3 CertificationReport.

**Compliance Policy (ace-policy.yang):** A YANG-modeled set of rules, thresholds, and ECA definitions against which AUT behavior is evaluated during Layer 1 certification. See Section 5.2.

**Compliance Test Suite (CTS):** A collection of test cases selected by ACE-NET based on the agent's `autonomy_level_claim` and declared interfaces.

**Network Function (NF):** A functional block within a network infrastructure, such as the 5G Core AMF, SMF, or UPF.

**ECA (Event-Condition-Action):** A policy rule pattern where a network event triggers evaluation of a condition and, if met, execution of a defined action. Used to validate autonomous remediation decisions in Layer 1.

**YANG:** A data modeling language (RFC 7950) used in ACE-NET for compliance policy (`ace-policy.yang`) and audit trail (`ace-audit.yang`) schemas.

### Evaluation Dimensions (IG1252)

An agent's autonomy behavior is assessed across five orthogonal dimensions defined in TMForum IG1252:

**Execution Dimension:** Observes whether the agent performs intended actions (remediation, provisioning, optimization) correctly and completely. Layer 1 certification measures execution via behavioral rules: does the agent issue correct TMF API calls in the right sequence?

**Awareness Dimension:** Observes whether the agent detects network state changes and faults. Layer 1 certification measures awareness by injecting faults and observing Time-to-Detect (TTD) metrics; TTD ≤ threshold indicates adequate awareness.

**Analysis Dimension:** Observes whether the agent correctly diagnoses root causes and makes sound decisions. Layer 1 certification evaluates analysis by asserting ECA rule conditions — if the condition evaluates true (diagnosis correct), the action must execute (decision sound).

**Decision Dimension:** Observes whether the agent chooses appropriate remediation actions given diagnosed state. Layer 1 certification validates decision-making by comparing chosen TMF API sequences against policy-defined correct sequences for each fault scenario.

**Intent/Experience Dimension:** Observes whether end-to-end network operator intent is fulfilled given the agent's decisions. Layer 3 certification evaluates intent fulfillment: each HVS scenario declares an intent (e.g., "resolve fault within 600 seconds"), and ACE-NET verifies that Time-to-Remediate (TTR) and all post-remediation metrics meet declared thresholds.

### Autonomous Network Effectiveness Indicators (ANEIs, IG1256)

The TMForum IG1256 specification defines a reference set of Key Effectiveness Indicators (KEIs) used to quantify autonomy level achievement. ACE-NET Layer 1 and Layer 3 certification measure the following ANEIs:

**Service Availability (%):** Fraction of time the target service is operational (not down). Threshold targets: L0: Not measured; L1: 95%; L2: 97%; L3: 99%; L4: 99.5%.

**Mean Time to Detect (seconds):** Average time interval from fault injection to detection. TTD is derived from Phase 0 bucketed events: (first detection timestamp) - (fault injection timestamp). Thresholds: L0: Not measured; L1: 3600s; L2: 300s; L3: 30s; L4: 5s.

**Mean Time to Repair (seconds):** Average time interval from detection to service restoration. Thresholds: L0: Not measured; L1: 1800s; L2: 600s; L3: 120s; L4: 30s.

**Incident Reduction Rate (%):** Fraction of recurring faults prevented by autonomous action. Computed post-certification by tracking recurrence of faults against which the agent was certified.

**Automation Ratio (%):** Fraction of operator-declared remediation actions executed automatically (vs. manual). Threshold targets: L0: 0%; L1: 10%; L2: 40%; L3: 70%; L4: 95%.

**Intelligent Ratio (%):** Fraction of automated actions that required AI/ML reasoning (vs. static policy). Threshold targets: L0: 0%; L1: 0%; L2: 0%; L3: 20%; L4: 60%.

### Autonomy Maturity Levels (IG1218F)

TMForum IG1218F formalizes the autonomy progression as a five-level classification tied to evaluation dimension performance:

**L0 (Manual):** No automation. Operator explicitly commands all actions. Execution, Awareness, Analysis, Decision, Intent dimensions are all operator-driven.

**L1 (Assisted):** Event reporting and basic alerting. Agent detects faults (Awareness) and reports to operator; operator makes all decisions (Decision, Execution remain manual). Example: alert on TTD < 3600s.

**L2 (Partial Self-X):** Static policy-driven remediation. Agent detects faults (Awareness ✓), applies pre-defined actions (Decision ✓), executes via TMF APIs (Execution ✓), but cannot adapt policy. Applicable to single-domain, well-understood scenarios.

**L3 (Conditional Self-X):** Intent-driven with multi-agent coordination. Agent detects faults (Awareness ✓), analyzes root cause (Analysis ✓), decides adaptive remediation via A2A-T peer agents (Decision ✓), executes within single domain (Execution ✓). Intent fulfillment requires operator-declared thresholds met.

**L4 (High Self-X):** AI/ML-driven cross-domain orchestration. Agent exhibits full autonomy across all five dimensions. Learns from past outcomes to improve future decisions. Coordinates multi-domain workflows (Agent Fabric) to fulfill operator intent.

### Key Effectiveness Indicator (KEI) Hierarchy (IG1218F)

The TMForum IG1218F framework defines a three-tier hierarchy for effectiveness measurement:

**Key Business Indicator (KBI):** Top-level business outcome metric set by the operator. Examples: "Network downtime < 0.1% per month", "Customer churn reduction > 15%". KBIs drive certification scope definition.

**Key Effectiveness Indicator (KEI):** Intermediate technical metric that directly supports KBI achievement. Example: To support "downtime < 0.1%", the KEI is "Mean Time to Repair (MTTR) < 120 seconds". The ANEIs defined in IG1256 (Service Availability, MTTR, MTBF, etc.) form the normative KEI reference set.

**Key Capability Indicator (KCI):** Leaf technical metric that measures autonomy dimension performance. The two primary KCIs are Automation Ratio (% of actions automated) and Intelligent Ratio (% of automated actions using AI/ML). Together, KCIs reflect the agent's autonomy maturity.
