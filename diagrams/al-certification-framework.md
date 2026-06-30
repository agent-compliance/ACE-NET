# TMForum Autonomous Networks — ACE-NET Certification Level Map

Maps TMForum autonomy levels L0–L4 (IG1218 / IG1401 Level 4 Industry Blueprint) to ACE-NET certification tiers, test suite types, and required TMF Open APIs. A2A-T protocol compliance (IG1453) is mandatory from L3 upward.

```mermaid
flowchart LR
    classDef l0 fill:#FFE0E0,stroke:#CC0000,color:#000000
    classDef l1 fill:#FFE8CC,stroke:#CC6600,color:#000000
    classDef l2 fill:#FFFFF0,stroke:#997700,color:#000000
    classDef l3 fill:#E8FFE8,stroke:#006600,color:#000000
    classDef l4 fill:#D0E8FF,stroke:#003399,color:#000000
    classDef note fill:#FAFAFA,stroke:#BBBBBB,color:#444444

    L0["L0 — Manual<br/>Scope: None<br/>Human: All decisions<br/> <br/>Tier 0 · Interface conformance<br/>TMF: TMF639<br/>3GPP: TS 28.620"]:::l0

    L1["L1 — Assisted<br/>Scope: Single task, guided<br/>Human: Approves every action<br/> <br/>Tier 1 · Behavioral certification<br/>Recommendation accuracy,<br/>telemetry fidelity, alert quality<br/>TMF: TMF628, TMF688<br/>3GPP: TS 28.530"]:::l1

    L2["L2 — Partial Self-X<br/>Scope: Bounded single task<br/>Human: Approves exceptions only<br/> <br/>Tier 2 · Playbook compliance cert<br/>Playbook correctness,<br/>boundary conditions, rollback<br/>TMF: TMF640, TMF654<br/>3GPP: TS 28.541"]:::l2

    L3["L3 — Conditional Self-X<br/>Scope: Multi-task, single domain<br/>Human: Sets policy, handles edges<br/> <br/>Tier 3 · A2A-T protocol cert<br/>+ domain behavior cert<br/>IG1453 conformance,<br/>single-domain HVS, ECA policy<br/>TMF: 640,641,654,628<br/>O-RAN: O1, A1, E2"]:::l3

    L4["L4 — High Self-X<br/>Scope: Cross-domain, intent-driven<br/>Human: Sets intents and SLAs only<br/> <br/>Tier 4 · Agent Fabric cert<br/>+ HVS scenario cert<br/>Multi-agent A2A-T workflows,<br/>cross-domain HVS, intent fulfillment<br/>TMF: 640,641,654,688,628,639,724<br/>IG1453 full Fabric · IG1401 L4 target"]:::l4

    N3["A2A-T IG1453<br/>required from L3 up<br/>Agent Card · Task lifecycle<br/>IG1453A prompt meta-model"]:::note

    N4["IG1401 Level 4 Target State<br/>Agent Fabric runtime C26.0.910<br/>Registry Center + Orch Center<br/>required for cert"]:::note

    L0 -->|evolve| L1
    L1 -->|evolve| L2
    L2 -->|evolve| L3
    L3 -->|evolve| L4
    N3 -.-> L3
    N4 -.-> L4
```

## Level Summary

| Level | Label | ACE-NET Tier | Chaos Faults in Scope |
|-------|-------|-------------|----------------------|
| L0 | Manual | Tier 0 — Interface conformance | None |
| L1 | Assisted | Tier 1 — Behavioral | None |
| L2 | Partial Self-X | Tier 2 — Playbook compliance | Network layer only |
| L3 | Conditional Self-X | Tier 3 — A2A-T protocol + domain | Network + single-domain fabric |
| L4 | High Self-X | Tier 4 — Fabric + HVS scenario | Network + full Agent Fabric layer |
