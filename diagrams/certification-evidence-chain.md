# ACE-NET Three-Layer Certification Evidence Chain

Full certification pipeline sequence across three layers: Layer 1 (behavioral — isolated agent under network chaos), Layer 2 (A2A-T protocol compliance per IG1453), and Layer 3 (end-to-end HVS scenario intent fulfillment). Culminates in a JWT Compliance Certificate that explicitly states the autonomy level achieved, which may differ from the level claimed.

```mermaid
sequenceDiagram
    actor Op as Operator
    participant Core as ACE-NET Core Engine
    participant Policy as Policy Engine
    participant Evid as Evidence Store
    participant TEAN as TEA Network Plugins
    participant TEAA as TEA A2A-T Plugin

    box LightGreen Agent Fabric IG1453
        participant Reg as Registry Center
        participant Orch as Orchestration Center
        participant AUT as Agent Under Test
    end

    Op->>Core: Submit { agentProfile,<br/>autonomy_level_claim: L4,<br/>hvs_scenarios: [HVS-1, HVS-5],<br/>a2at_endpoint: https://... }
    Core->>Policy: LoadPolicies(agent_type, domain_scope)
    Policy-->>Core: PolicySet { rules, thresholds, ECA definitions }

    rect rgb(220, 235, 255)
        Note over Op,AUT: LAYER 1 — Behavioral Certification (isolated agent + network chaos)
        Core->>TEAN: ProvisionSandbox(testSpec)
        TEAN-->>Core: sandbox_ready
        Core->>TEAN: DeployAUT(artifact_url)
        TEAN->>AUT: deploy and configure
        AUT-->>TEAN: ready { agentEndpoint }
        Core->>TEAN: ApplyChaosSequence [NF_crash AMF,<br/>link_degrade backhaul,<br/>resource_cap SMF CPU 90pct,<br/>slice_SLA_breach latency]
        TEAN->>AUT: fault propagation via network
        AUT-->>TEAN: telemetry stream { PM counters,<br/>alarm events, API calls, ECA decisions }
        TEAN->>Core: BehaviorStream real-time
        Core->>Policy: EvaluateRules(observed_behavior)
        Policy-->>Core: RuleResults { evaluated: 24, passed: 21, failed: 3, score: 87.5 }
        Core->>Evid: StoreEvidence(layer=1, results, traces)
        Core->>TEAN: TeardownSandbox()
        Note right of Core: Layer 1 fragment<br/>behavioral_score: 87.5<br/>rules: 21/24 passed<br/>TMF API conformance: PASS
    end

    rect rgb(220, 255, 220)
        Note over Op,AUT: LAYER 2 — A2A-T Protocol Certification (IG1453 conformance)
        Core->>TEAA: ValidateAgentCard(aut_base_url)
        TEAA->>AUT: GET /.well-known/agent.json
        AUT-->>TEAA: AgentCard { skills, authSchemes, endpoints }
        TEAA->>Reg: RegisterAgent(AgentCard)
        Core->>TEAA: RunProtocolSuite [AgentCard, TaskLifecycle,<br/>SubTaskDelegation, FaultHandling, PromptMetaModel]
        TEAA->>AUT: POST /tasks { IG1453A structured prompt }
        AUT-->>TEAA: { taskId, status: submitted }
        AUT->>Reg: ResolveAgent(skill=billing-check)
        Reg-->>AUT: peer_endpoint → TEAA stub
        AUT->>TEAA: POST /tasks { sub-task }
        TEAA-->>AUT: sub-task artifact
        Core->>TEAA: InjectFabricFaults [registry_unavailable,<br/>task_silent_drop, malformed_IG1453A,<br/>illegal_state_transition]
        TEAA-->>Core: ProtocolTestResults { lifecycle: PASS,<br/>multi_agent: PASS, error_handling: PASS,<br/>prompt_meta_model: PASS, fabric_faults: 4/4 }
        Core->>Evid: StoreEvidence(layer=2, ig1453_conformance_log)
        Note right of Core: Layer 2 fragment<br/>ig1453: v1.0.0 PASS<br/>agent_card_valid: true<br/>skills_certified: [complaint-triage]<br/>fabric_fault_resilience: 4/4
    end

    rect rgb(255, 248, 220)
        Note over Op,AUT: LAYER 3 — HVS Scenario Certification (end-to-end intent)
        Core->>Orch: InitiateHVS { scenario: HVS-5 ComplaintHandling,<br/>intent: resolve complaint in under 10 min }
        Orch->>AUT: trigger scenario via A2A-T dispatch
        AUT->>Reg: ResolveAgent(skill=network-check)
        AUT->>Reg: ResolveAgent(skill=billing-check)
        AUT->>Reg: ResolveAgent(skill=crm-update)
        Reg-->>AUT: peer endpoints → TEAA stubs
        AUT->>TEAA: sub-task network-check
        TEAA-->>AUT: { network: OK }
        AUT->>TEAA: sub-task billing-check
        TEAA-->>AUT: { billing: anomaly_detected, overcharge: true }
        AUT->>TEAA: sub-task crm-update
        TEAA-->>AUT: { crm: ticket_updated, credit_issued }
        Core->>TEAA: InjectFabricFault(orchestration_timeout)
        Note over Orch: Orchestration timeout injected
        AUT->>TEAN: EscalateToHuman(reason=orchestration_unavailable)
        Note right of AUT: Correct L3 behavior — escalate<br/>when orchestration unavailable
        AUT-->>Orch: ResolutionReport { root_cause: billing,<br/>actions: [credit_issued, ticket_closed],<br/>elapsed: 7m 18s }
        Core->>Policy: EvaluateIntentFulfillment { sla: under 10 min, actual: 7m 18s }
        Policy-->>Core: IntentResult { fulfilled: true, margin: 2m 42s }
        Core->>Evid: StoreEvidence(layer=3, scenario_trace, intent_sla_proof)
        Note right of Core: Layer 3 fragment<br/>HVS-5: PASS, elapsed 7m 18s<br/>HVS-1: FAIL cross-domain incomplete<br/>intent_fulfillment: 0.73
    end

    Core->>Evid: AssembleCertificate()
    Evid-->>Core: CertificatePayload

    Note over Core: ACE-NET Compliance Certificate JWT<br/>autonomy_level_claimed: L4<br/>autonomy_level_ACHIEVED: L3 — HVS-1 failed<br/>layer1 behavioral_score: 87.5<br/>layer2 ig1453_conformance: PASS<br/>layer3 hvs_passed: [HVS-5] failed: [HVS-1]<br/>intent_fulfillment_rate: 0.73<br/>issued_by: ACE-NET-CA

    Core-->>Op: ComplianceCertificate JWT + EvidenceReport PDF
```

## Certification Layer Summary

| Layer | Scope | Passes When | Key Evidence |
|-------|-------|-------------|-------------|
| **Layer 1 — Behavioral** | Isolated agent under network chaos | Behavioral score ≥ threshold, rules pass rate ≥ threshold | PM counters, alarm logs, API call traces |
| **Layer 2 — A2A-T Protocol** | IG1453 protocol conformance | All protocol assertions pass | Agent Card, task logs, fault handling traces |
| **Layer 3 — HVS Scenario** | End-to-end intent fulfillment | Scenario SLA met, intent fulfilled | Scenario trace, SLA proof, escalation log |

> **Note:** `autonomy_level_achieved` is computed by ACE-NET from Layer 3 results and may be lower than `autonomy_level_claimed`. An agent claiming L4 that fails cross-domain HVS-1 receives an L3 certificate.
