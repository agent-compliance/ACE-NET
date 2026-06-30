## 3. ACE-NET Architectural Framework

ACE-NET operates as an independent certification authority that overlays the telecom agent execution environment without participating in its operational data path. The architecture is aligned to the TMForum IG1401 Level 4 Industry Blueprint and the IG1453 A2A-T Agent Fabric specification (Catalyst C26.0.910).

### 3.1. Four-Plane Reference Architecture

The framework is organized into four functional planes. The full reference architecture is illustrated in Figure 1 (see `diagrams/ace-architecture.md`).

**Business / Intent Plane:** Carries operator intent expressed through natural language or structured SID Intent Models, translated into agent tasks. OSS/BSS components expose TMF Open APIs as the northbound interface for agent interaction: TMF628 (Performance Mgmt), TMF640 (Activation & Config), TMF641 (Service Order), TMF654 (Trouble Ticket), TMF688 (Event Mgmt), and TMF724 (AI/ML Model Mgmt).

**Agent Fabric Plane:** Implements the A2A-T inter-agent protocol as specified in IG1453. The Registry Center manages Agent Card publication, skill discovery, and authentication (OAuth2 or mTLS). The Orchestration Center coordinates multi-agent workflows using the IG1453A Prompt Meta-Model. Domain Agents operate within this plane: RAN Agent (O-RAN xApp/rApp), Core NF Agent (AMF/SMF/UPF lifecycle), Slice Agent (E2E slice management), Assurance Agent (SLA/fault management), and OSS/BSS Agent (order/ticket handling).

**Network Infrastructure Plane:** Contains the physical and virtual network elements across three domains:
- O-RAN: Non-RT RIC (A1 policy), Near-RT RIC (E2 control), O-DU/O-RU
- 5G Core: AMF/SMF/UPF, NRF/PCF/NSSF, per 3GPP TS 28.xxx series
- Transport: IP/MPLS/Optical fabric managed via NETCONF/YANG (RFC 6241)

**ACE-NET Certification Plane:** The Core Engine, Policy Engine, Evidence Store, and Telecom Environment Adapter (TEA) constitute the certification infrastructure. The Certification Plane overlays all three operational planes via TEA plugins (Section 3.2) without altering their operational state.

An operator or CI/CD system submits an Agent Profile with an `autonomy_level_claim` to the Core Engine; upon completion the Core Engine returns a signed Compliance Certificate JWT (Section 4.5) to the operator.

### 3.2. Telecom Environment Adapter (TEA)

The TEA abstracts the heterogeneous interfaces of the telecom ecosystem into a uniform command surface for the ACE-NET Core Engine. Six plugin types are defined:

| Plugin | Interface Standard | Scope |
|--------|-------------------|-------|
| 3GPP NBI Plugin | 3GPP TS 28.530 | 5G Core NF lifecycle, PM, fault management |
| O-RAN O1/A1/E2 Plugin | O-RAN WG2/WG3 | Non-RT RIC, Near-RT RIC, xApp control loop |
| NETCONF/YANG Plugin | RFC 6241, RFC 7950 | Transport fabric configuration |
| TMF OpenAPI Plugin | TMF OpenAPI v4 | OSS/BSS service, ticket, PM APIs |
| A2A-T Plugin | IG1453 v2.1.0 | Agent Card validation, task intercept, fabric fault injection |
| Chaos Injector | Internal | Network-layer and Agent Fabric-layer fault injection |

The **A2A-T Plugin** is required for Tier 3 and Tier 4 certification. It performs three functions: (1) validating Agent Card publication at `{a2at_endpoint}/.well-known/agent.json` per IG1453 §4, including schema validation per the Agent Card specification with identity, version, capabilities array (each with capability-name, version, input-schema, output-schema), and constraints; (2) acting as a peer agent stub for sub-task delegation testing, registering stubs in the TEA Registry Center for each skill declared in `scenario_profile_ids`; and (3) injecting Agent Fabric faults defined in Section 9 (see `diagrams/chaos-fault-taxonomy.md`).

The **Task Lifecycle** in IG1453 v2.1.0 defines an explicit state machine (PENDING → ASSIGNED → RUNNING → COMPLETED/FAILED) for each sub-task delegation. Layer 2 Phase A validates Agent Card publication; Phase B validates correct state transitions during multi-agent delegation; Phase C validates error handling (e.g., task timeout, peer failure, contract violation); Phase D validates IG1453A Prompt Meta-Model compliance (structured task representation and response validation).

The **Chaos Injector** applies fault sequences to both the Network Infrastructure Plane (from Tier 2 upward) and the Agent Fabric Plane (from Tier 3 upward). Network-layer and Agent Fabric-layer fault categories are defined in Section 9.

### 3.3. Three-Layer Certification Model

ACE-NET certifies autonomous agents through three cumulative evidence layers. Each layer produces a certificate fragment; all three are assembled into the final Compliance Certificate (Section 4.5). The full sequence is illustrated in Figure 3 (see `diagrams/certification-evidence-chain.md`).

**Layer 1 — Behavioral Certification:** The AUT is deployed in an isolated sandbox. The Chaos Injector applies a tier-appropriate network-layer fault sequence. Observed behavior is evaluated against the `ace-policy.yang` rule set (Section 5.2). A behavioral score (rules_passed / rules_evaluated × 100) and TMF API conformance result are produced.

**Layer 2 — A2A-T Protocol Certification:** Executed only when `autonomy_level_claim` >= L3. The A2A-T Plugin validates IG1453 conformance across four phases: Agent Card validation, task lifecycle state machine, sub-task delegation via Registry-based discovery, and fault and error handling (see `diagrams/a2at-compliance-flow.md`). Layer 2 produces a binary PASS/FAIL.

**Layer 3 — HVS Scenario Certification:** Executed only when `autonomy_level_claim` >= L3. The Orchestration Center dispatches the HVS scenarios declared in `scenario_profile_ids` (Section 8). The AUT must fulfill each scenario's declared intent within the scenario SLA. Partial scenario failure reduces the achieved autonomy level.

Layers execute sequentially. A Layer 1 score below threshold halts certification and issues a certificate at the highest achievable tier from Layer 1 results alone.

### 3.4. Evaluation Dimension Binding

The five evaluation dimensions defined in IG1252 (Execution, Awareness, Analysis, Decision, Intent/Experience) map to ACE-NET's three-layer certification model as follows:

**Layer 1** binds Execution, Awareness, and Analysis dimensions:
- **Execution** is measured via TMF API conformance: ACE-NET validates that all tool invocations use correct schema, semantics, and call sequencing per the TMF Open API specifications referenced in the Agent Profile interfaces.
- **Awareness** is measured via Time-to-Detect (TTD) metrics: The Chaos Injector injects faults; the TEA trace pipeline (ACE Phase 0) extracts TTD from bucketed events. TTD ≤ policy threshold indicates the agent detected the fault in a timely manner.
- **Analysis** is measured via ECA rule evaluation: Policy-defined rules encode correct root-cause analysis for each fault scenario (condition block). When a fault is injected and the agent observes it, the Policy Engine evaluates the corresponding rule's condition. If the condition evaluates true, the agent's internal model (and hence analysis) is deemed correct.

**Layer 2** binds the Decision dimension:
- **Decision** is measured via A2A-T protocol compliance: The layer validates that the agent correctly decides whether to delegate a sub-task to a peer agent (via Registry lookup) vs. execute locally. Multi-agent delegation is tested via the TEA Registry Center and Orchestration Center stubs. Incorrect delegation decisions (e.g., attempting to execute a remote-only capability locally) cause Layer 2 failure.

**Layer 3** binds the Intent/Experience dimension:
- **Intent/Experience** is measured via HVS scenario fulfillment: Each HVS scenario declares an operator intent (e.g., "resolve critical fault and restore service to 99.5% availability within 600 seconds"). The Orchestration Center dispatches the scenario; the agent must fulfill the intent within declared SLA. Partial or failed intent fulfillment results in HVS scenario failure.

An agent that passes all Layer 1 dimension assessments but fails Layer 2 Decision or Layer 3 Intent measurements receives an achieved autonomy level one tier lower than claimed. This ensures that claimed autonomy is backed by evidence across all five dimensions.

### 3.5. Autonomy Level and Certification Tier Mapping

ACE-NET certification tiers map directly to the TMForum IG1218 Autonomous Networks autonomy levels and the IG1401 Level 4 Industry Blueprint target state. Figure 2 illustrates this mapping (see `diagrams/al-certification-framework.md`).

| TMForum Level | Label | ACE-NET Tier | A2A-T Required | HVS Scope |
|---------------|-------|-------------|----------------|-----------|
| L0 | Manual | Tier 0 — Interface conformance | No | None |
| L1 | Assisted | Tier 1 — Behavioral | No | None |
| L2 | Partial Self-X | Tier 2 — Playbook compliance | No | None |
| L3 | Conditional Self-X | Tier 3 — A2A-T protocol + domain | Yes (IG1453) | Single-domain |
| L4 | High Self-X | Tier 4 — Agent Fabric + HVS | Yes (full Fabric) | Cross-domain |

An agent MUST meet all requirements of all lower tiers before claiming a higher tier. An agent that claims L4 but fails a required cross-domain HVS scenario receives a Compliance Certificate with `autonomy_level_achieved` set to the highest tier for which all requirements are met (Section 4.5).

### 3.6. Key Components

**ACE-NET Core Engine:** Manages the certification lifecycle — parsing the Agent Profile, selecting test suites based on `autonomy_level_claim`, orchestrating the three certification layers, and signing the Compliance Certificate JWT.

**Policy Engine:** Evaluates observed behavior against the `ace-policy.yang` rule set (Section 5.2). Supports metric-based thresholds, state assertions, API call sequence validation, and Event-Condition-Action (ECA) definitions. The Policy Engine operates in real time during Layer 1 and post-execution during Layer 3.

**Evidence Store:** Assembles the three-layer audit trail into an `ace-audit.yang` instance (Section 5.3). Produces the final CertificatePayload signed by the ACE-NET Certificate Authority.

**Registry Center (IG1453):** Provides agent discovery, authentication, and skill management across the Agent Fabric. The A2A-T Plugin intercepts Registry interactions during Layer 2 and Layer 3 certification to validate IG1453 conformance without modifying production Registry state.

**Orchestration Center (IG1453):** Coordinates multi-agent workflows using the IG1453A Prompt Meta-Model. Used in Layer 3 HVS scenario execution and as an injection point for orchestration-level fault scenarios (e.g., `orchestration_timeout`, circular delegation loop).
