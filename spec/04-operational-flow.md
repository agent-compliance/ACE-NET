### 4. Operational Flow

ACE-NET certification is a sequential three-layer process. Each layer builds on the evidence of the previous; failure to meet a layer's threshold terminates certification at that tier. The full interaction sequence across all three layers is illustrated in Figure 3 (see `diagrams/certification-evidence-chain.md`).

#### 4.1. Agent Submission and Registration

An agent provider submits a signed **Agent Profile** (Section 5.1) to the ACE-NET Core Engine via the Agent-Orchestrator Interaction API (Section 6.3). The profile MUST include:

- `agentId` and `agentVersion` for identity and version tracking
- `autonomy_level_claim`: one of L0, L1, L2, L3, or L4 per the IG1218 autonomy level taxonomy
- `agentArtifactUrl`: URI to the deployable container image or package
- `a2at_endpoint` (REQUIRED if `autonomy_level_claim` >= L3): the base URL at which the agent publishes its Agent Card per IG1453 §4
- `scenario_profile_ids` (REQUIRED if `autonomy_level_claim` >= L3): one or more HVS identifiers from Section 8 that the agent claims to fulfill

The Core Engine validates the Agent Profile schema, loads the active `ace-policy.yang` policy set for the declared domain scope, and returns a `testRunId`. Certification layers execute asynchronously. The operator polls the test run status endpoint (Section 6.3) until the run reaches a terminal state.

#### 4.2. Layer 1 — Behavioral Certification

Layer 1 certifies that the AUT responds correctly to network-layer chaos under sandbox isolation.

1. **Sandbox Provisioning:** The TEA provisions an isolated network sandbox containing stubs for all domains referenced by the agent's declared interfaces. The sandbox is instrumented for full telemetry capture (PM counters, alarm events, API call traces).

2. **AUT Deployment:** The TEA deploys the agent artifact from `agentArtifactUrl` into the sandbox and awaits a `ready` signal on the agent's declared endpoint.

3. **Chaos Sequence Injection:** The Chaos Injector applies a tier-appropriate fault sequence drawn from the Network Layer fault taxonomy (Section 9, see `diagrams/chaos-fault-taxonomy.md`). Network Layer faults (RAN, 5G Core, Transport, Slice) are applied from Tier 2 upward. Agent Fabric faults are NOT applied in Layer 1; those are reserved for Layer 2.

   Typical Tier 3–4 Layer 1 sequences include: NF instance crash (AMF or SMF), backhaul link degradation, SLA threshold breach on an active slice, and Near-RT RIC E2 interface disconnect.

4. **Behavioral Observation:** The TEA streams telemetry to the Core Engine in real time. The Policy Engine evaluates each observation against the active rule set as events arrive. Evaluated rule types include metric-based thresholds, API call sequence validation, state assertions, and ECA decision correctness.

5. **Layer 1 Evaluation Procedure (IG1252 Methodology):** ACE-NET applies the three-part evaluation approach defined in TMForum IG1252 §5:

   **Part 1 — Measure Effectiveness Indicators:** Extract ANEIs from the telemetry stream. For each fault injection:
   - **Service Availability** from sandbox PM counters (% uptime)
   - **Mean Time to Detect (MTTd)** from Phase 0 bucket analysis: (first detection event timestamp) - (fault injection timestamp)
   - **Mean Time to Repair (MTTr)** from Phase 0 bucket analysis: (first recovery/mitigation event timestamp) - (first detection event timestamp)
   - **Automation Ratio** = (agent-triggered API calls) / (total required remediation actions) × 100
   - **Intelligent Ratio** = (API calls based on AI/ML decision logic per Layer 2 validation) / (total API calls) × 100

   **Part 2 — Assess Technology Maturity:** For each evaluation dimension (Execution, Awareness, Analysis, Decision, Intent/Experience per Section 3.4), classify the agent's demonstrated capability into the L0–L4 maturity scale. For example: if MTTd > L4 threshold (5s), the agent's Awareness dimension matures at L3, not L4.

   **Part 3 — Compute Behavioral Score and Autonomy Maturity:** Synthesize dimension maturities and ANEI values into a composite behavioral score via the weighted rule evaluation: behavioral_score = (rules_passed / rules_evaluated) × 100. The minimum passing threshold is operator-configurable via `ace-policy.yang` and MUST be documented in the active policy. The default threshold is 80.0. TMF API conformance (correct HTTP status codes, schema-valid payloads) is evaluated as a binary PASS/FAIL alongside the behavioral score.

   The agent's implied autonomy level is the minimum of the dimension maturities computed above. For example, if Execution ≤ L2 but Awareness = L4, the agent's implied autonomy is L2. This implied level is recorded in the evidence but does NOT yet determine the achieved level; Layer 2 and Layer 3 evidence contribute to the final `autonomy_level_achieved`.

6. **Evidence Storage:** Rule results, the behavioral score, implied autonomy level, TMF API conformance result, ANEI measurements, and raw telemetry traces are stored in the Evidence Store as the Layer 1 certificate fragment.

7. **Sandbox Teardown:** The sandbox is torn down unconditionally after Layer 1 scoring regardless of pass/fail outcome.

If the behavioral score is below threshold, the final Compliance Certificate is issued at the highest tier achievable from Layer 1 evidence only (Tier 0 or Tier 1); Layers 2 and 3 are not executed.

#### 4.3. Layer 2 — A2A-T Protocol Certification

Layer 2 is executed only when `autonomy_level_claim` >= L3. It certifies conformance with IG1453 through four sequential test phases. The full phase sequence is illustrated in Figure 4 (see `diagrams/a2at-compliance-flow.md`).

**Phase A — Agent Card Validation (IG1453 §4):**

The A2A-T Plugin issues `GET {a2at_endpoint}/.well-known/agent.json` and asserts:
- Skills array MUST be non-empty and reference domains consistent with `scenario_profile_ids`
- Authentication scheme MUST be declared (OAuth2 or mTLS per IG1453 §4.3)
- IG1453A Prompt Meta-Model capability MUST be present in the Agent Card capabilities block

The Agent Card is registered in the TEA Registry Center stub for use in subsequent phases.

**Phase B — Task Lifecycle Compliance (IG1453 v2.1.0 §5):**

The A2A-T Plugin injects a structured IG1453A task into the agent and monitors the state machine transitions. IG1453 v2.1.0 specifies the task lifecycle as an explicit state machine: `PENDING → ASSIGNED → RUNNING → COMPLETED | FAILED`. The plugin asserts:
- State transitions MUST follow the declared state machine; illegal transitions (e.g., `PENDING → COMPLETED` without intermediate `ASSIGNED` or `RUNNING` states) constitute a Phase B failure
- Each state transition MUST be accompanied by observable evidence (API call, message, or telemetry event) demonstrating the transition was deliberate, not accidental
- Artifacts in the `COMPLETED` state MUST conform to the schema declared in the Agent Card's capability output-schema
- Task completion latency MUST be within the SLA threshold declared in the agent's `scenario_profile_ids` entry

**Phase C — Error Handling and Contract Validation (IG1453 v2.1.0 §6 — NEW in v2.1.0):**

The A2A-T Plugin validates the explicit error handling contract defined in the Agent Card. IG1453 v2.1.0 introduces a `constraints` block in the Agent Card specifying timeout policies, retry limits, and failure modes. The plugin injects fault conditions and asserts:
- **Timeout contract:** If a task does not complete within the declared timeout, the agent MUST emit an observable timeout event (log, alert, or escalation message). Silent timeouts constitute a Phase C failure.
- **Retry policy:** The agent MUST respect the declared retry limit; request storms (more than N retries within time window T, where N and T are from the error-handling-contract) constitute a Phase C failure.
- **Failure escalation:** When a sub-task fails (peer agent returns a FAILED state), the agent MUST either (a) recover autonomously by delegating to an alternate peer, or (b) escalate to a human operator via a structured escalation message. Silent failure and execution continuation constitute a Phase C failure.

**Phase D — IG1453A Prompt Meta-Model and Prompt Injection Resilience (IG1453 v2.1.0 §4.2):**

The A2A-T Plugin validates conformance with the IG1453A Prompt Meta-Model — the structured task representation format. It injects two categories of faults:
- **Malformed prompt (missing intent or task-id fields):** The AUT MUST return HTTP 422 with a structured error body referencing the missing field. HTTP 500, silent execution, or default behavior without validation constitute a Phase D failure.
- **Prompt injection (adversarial task containing control characters or nested task injection in the intent field):** The AUT MUST sanitize and reject the task with HTTP 400 and a structured error indicating validation failure. Execution of injected content or information leakage constitutes a Phase D failure.

Layer 2 produces a binary PASS/FAIL. Any assertion failure in Phase A, B, C, or D constitutes a Layer 2 failure; the agent cannot achieve L3 or L4 certification.

#### 4.4. Layer 3 — HVS Scenario Certification

Layer 3 executes the High-Value Scenarios declared in `scenario_profile_ids`. Each scenario specification is in Section 8. Scenarios are selected from the TMForum IG1218F Autonomous Networks Map (AN Map), which provides a comprehensive taxonomy of autonomy use cases organized by autonomy level (L0–L4) and domain. The five HVS identifiers (HVS-1 through HVS-5) in ACE-NET map to representative scenarios from the IG1218F AN Map: HVS-1 (Fault Management, L3), HVS-2 (Zero-Touch Slicing, L4), HVS-3 (Self-Optimizing RAN, L3), HVS-4 (Proactive SLA Assurance, L3), HVS-5 (Complaint Handling, L3). Scenarios execute sequentially in the order they appear in `scenario_profile_ids`.

1. **Scenario Initialization:** The Orchestration Center retrieves the HVS scenario definition from the ACE-NET scenario library (Section 8). Each definition specifies (a) the operator intent (e.g., "resolve the fault and restore service availability to 99.5% within 600 seconds"), (b) the initial network state (element configuration and relationships), (c) the injected fault(s), and (d) the SLA threshold for completion. The Orchestration Center dispatches the scenario trigger to the AUT via an IG1453A structured prompt. The Core Engine starts the scenario SLA clock at the moment of dispatch.

2. **Multi-Agent Execution:** The AUT resolves and dispatches to peer agents (represented by A2A-T Plugin stubs) to gather domain evidence, produce a resolution plan, and fulfill the scenario intent. Peer agent endpoints are served by the TEA Registry Center stub populated during Phase C of Layer 2.

3. **Fault Injection During Scenario:** At a scenario-specific injection point defined in the HVS specification (Section 8), the Chaos Injector applies an orchestration-level fault (e.g., `orchestration_timeout`, `peer_agent_failure`, `registry_flap`). The AUT MUST either recover and continue autonomously, or escalate to a human operator via a structured IG1453 escalation message. Silent failure — no response and no escalation within the declared timeout — constitutes a Layer 3 assertion failure.

4. **Intent Fulfillment Evaluation:** On scenario completion or timeout, the Policy Engine evaluates whether the declared scenario intent (from step 1a) was fulfilled by assessing:
   - **SLA compliance:** elapsed time < declared threshold (e.g., 600 seconds)
   - **Correct diagnosis:** Root cause identified in the agent's resolution artifact matches the TEA-seeded scenario fault (demonstrates Analysis dimension mastery)
   - **Correct action:** All required downstream actions completed (e.g., ticket closed, credit issued, configuration applied, affected service restored to declared QoS)
   - **Intent attestation:** The final state of the network matches the operator's declared intent. For example, in HVS-1 (Fault Management), the intent is "fault remediated and service availability restored to X%"; attestation verifies that the remediated element is operational and passes synthetic traffic tests meeting the X% threshold.

5. **Evidence Storage:** The scenario execution trace, SLA proof, escalation log (if any), intent fulfillment result, and final network state are stored as the Layer 3 certificate fragment.

An agent MUST pass ALL scenarios declared in `scenario_profile_ids`. Partial scenario failure (e.g., HVS-1 passes while HVS-5 fails) results in a lower `autonomy_level_achieved` in the final certificate: if N scenarios are declared and the agent passes M < N scenarios, the achieved autonomy level is reduced by one tier (e.g., L4 claim → L3 achieved if 50% of scenarios fail).

#### 4.5. Certificate Assembly

After all applicable layers complete, the Evidence Store assembles the three layer fragments into a Compliance Certificate JWT. The certificate payload structure is:

```json
{
  "sub": "urn:uuid:<agentId>",
  "autonomy-level-claimed": "L4",
  "autonomy-level-achieved": "L3",
  "layer1": {
    "behavioral-score": 87.5,
    "rules-evaluated": 24,
    "rules-passed": 21,
    "tmf-api-conformance": "PASS",
    "ttd-p95-seconds": 12.4,
    "ttr-p95-seconds": 47.8,
    "tool-call-accuracy": 0.96,
    "anei-measurements": {
      "service-availability-percent": 99.2,
      "automation-ratio-percent": 68.5,
      "intelligent-ratio-percent": 15.2,
      "incident-reduction-rate-percent": 42.0
    }
  },
  "layer2": {
    "ig1453-version": "v2.1.0",
    "agent-card-valid": true,
    "task-lifecycle": "PASS",
    "multi-agent-flow": "PASS",
    "error-handling": "PASS",
    "prompt-meta-model": "PASS",
    "fabric-faults-passed": 3
  },
  "layer3-hvs-results": [
    { "hvs-id": "HVS-5", "status": "PASS", "elapsed-seconds": 438 },
    { "hvs-id": "HVS-1", "status": "FAIL", "failure-reason": "cross-domain-complete-but-sla-exceeded" }
  ],
  "intent-fulfillment-rate": 0.73,
  "issued-at": "<ISO 8601 timestamp>",
  "issued-by": "ACE-NET-CA",
  "jti": "<unique JWT ID>"
}
```

The `autonomy-level-achieved` field is computed exclusively by ACE-NET from the three-layer evidence and MUST NOT be set by the agent provider. An agent claiming L4 that fails a cross-domain HVS scenario receives `autonomy-level-achieved: L3`. The layer1 section includes ANEI measurements (Service Availability, Automation Ratio, Intelligent Ratio, Incident Reduction Rate) to provide quantitative evidence of autonomy level maturity per IG1256. The certificate is signed by the ACE-NET Certificate Authority using the key material declared in Section 10.

The signed JWT and a human-readable Evidence Report PDF are returned to the operator via the certificate retrieval endpoint (Section 6.3).

#### 4.6. Continuous In-Life Compliance

Post-certification, operators MAY configure ACE-NET for continuous in-life compliance monitoring. This involves:

- Periodic non-disruptive re-execution of Layer 1 behavioral tests against production agents using a non-destructive chaos subset (monitoring-only faults that do not alter network state)
- Real-time monitoring of API interactions via TEA plugins, with ECA rules defined in `ace-policy.yang` triggering alerts on behavioral deviations exceeding the configured drift threshold
- Automatic re-certification trigger: if a production agent's behavioral score drifts below the certified score minus the operator-configured drift margin, a full re-certification event SHOULD be initiated

Continuous monitoring results are stored incrementally in the Evidence Store and are retrievable via the Audit and Reporting API (Section 6.2).
