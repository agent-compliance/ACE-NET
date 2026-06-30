# A2A-T (IG1453) Protocol Compliance Test Flow — ACE-NET Layer 2 Certification

Sequence diagram for ACE-NET's Layer 2 certification: validating that an Agent Under Test (AUT) correctly implements the A2A-T protocol (IG1453) including Agent Card publication, task lifecycle state machine, multi-agent sub-task delegation, fault resilience, and IG1453A prompt meta-model compliance.

```mermaid
sequenceDiagram
    actor Core as ACE-NET Core
    participant TEA as TEA / A2A-T Plugin
    participant Reg as Registry Center
    participant Orch as Orchestration Center
    participant AUT as Agent Under Test

    rect rgb(220, 235, 255)
        Note over Core,AUT: Phase A — Agent Card Validation (IG1453 §4)
        Core->>TEA: ValidateAgentCard(aut_base_url)
        TEA->>AUT: GET /.well-known/agent.json
        AUT-->>TEA: AgentCard { name, skills[], authSchemes[], endpoints }
        Note right of Core: ASSERT: skills non-empty,<br/>auth scheme declared,<br/>IG1453A meta-model present
        Core->>Reg: RegisterAgent(AgentCard)
        Reg-->>Core: ack { agentId }
    end

    rect rgb(220, 255, 220)
        Note over Core,AUT: Phase B — Task Lifecycle Compliance (IG1453 §5)
        Core->>TEA: InjectTask(skill_id, IG1453A_prompt)
        TEA->>AUT: POST /tasks { id, sessionId, message }
        AUT-->>TEA: 202 Accepted { taskId, status: submitted }
        loop Poll until terminal state
            TEA->>AUT: GET /tasks/{taskId}
            AUT-->>TEA: { status: working | completed | failed, artifacts }
        end
        TEA-->>Core: FinalState { status, artifacts, timing }
        Note right of Core: ASSERT: submitted → working → completed<br/>no illegal transitions<br/>artifacts match declared schema<br/>latency within SLA threshold
    end

    rect rgb(255, 255, 220)
        Note over Core,AUT: Phase C — Sub-Task Delegation (A2A-T Multi-Agent)
        Core->>TEA: EnablePeerAgentStub(skills=[network-check, billing-check])
        TEA->>Reg: RegisterStubAgent(AgentCard_stub)
        Core->>TEA: InjectTask(skill=complaint-triage)
        TEA->>AUT: POST /tasks { complaint-triage task }
        AUT->>Reg: ResolveAgent(skill=network-check)
        Reg-->>AUT: peer_endpoint → TEA stub
        AUT->>TEA: POST /tasks { sub-task: network-check }
        TEA-->>AUT: { network: OK }
        AUT->>Reg: ResolveAgent(skill=billing-check)
        Reg-->>AUT: peer_endpoint → TEA stub
        AUT->>TEA: POST /tasks { sub-task: billing-check }
        TEA-->>AUT: { billing: anomaly_detected }
        AUT-->>TEA: FinalArtifact { root_cause: billing, resolution_plan }
        Note right of Core: ASSERT: AUT uses Registry, not hardcoded endpoints<br/>each sub-task has independent taskId<br/>AUT aggregates partial results correctly
    end

    rect rgb(255, 220, 220)
        Note over Core,AUT: Phase D — Fault and Error Handling (IG1453 §6)
        Core->>TEA: InjectFabricFault(target=Registry, type=unavailable)
        Note over Reg: Registry blocked
        Core->>TEA: InjectTask(requires peer resolution)
        TEA->>AUT: POST /tasks { new task }
        AUT->>Reg: ResolveAgent(...) — fails
        AUT-->>TEA: structured error response with backoff
        Note right of Core: ASSERT: exponential backoff, not storm<br/>structured error per IG1453 §6<br/>observable failure event emitted
        Note over Reg: Registry restored

        Core->>TEA: InjectMalformedPrompt(missing intent field)
        TEA->>AUT: POST /tasks { malformed IG1453A prompt }
        AUT-->>TEA: 422 { error: prompt schema invalid, field: intent }
        Note right of Core: ASSERT: reject malformed prompt, do not execute<br/>HTTP 422 not 500<br/>field reference in error body
    end

    Note over Core,AUT: Layer 2 Certificate Fragment
    Note right of Core: ig1453_version: v1.0.0<br/>agent_card_valid: true<br/>task_lifecycle: PASS<br/>multi_agent_flow: PASS<br/>error_handling: PASS<br/>prompt_meta_model: PASS
```

## Test Phase Summary

| Phase | What Is Tested | Key IG1453 Reference |
|-------|---------------|----------------------|
| A — Agent Card | Schema, skills taxonomy, auth scheme, IG1453A declaration | IG1453 §4.2 |
| B — Task Lifecycle | State machine correctness, artifact schema, SLA timing | IG1453 §5 |
| C — Sub-Task Delegation | Registry-based discovery, independent task IDs, result aggregation | A2A-T multi-agent flow |
| D — Fault Handling | Backoff on registry failure, graceful prompt rejection | IG1453 §6 / IG1453A §3 |
