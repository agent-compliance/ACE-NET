# ACE-NET Reference Architecture — A2A-T Aware

Four-plane reference architecture aligned to TMForum IG1401 Level 4 blueprint and IG1453 A2A-T Agent Fabric (Catalyst C26.0.910). The ACE-NET Certification Plane overlays all three operational planes via the Telecom Environment Adapter (TEA).

```mermaid
flowchart TD
    classDef actor  fill:#2E4057,stroke:#1a2d3f,color:#ffffff
    classDef biz    fill:#D0E8FF,stroke:#003399,color:#000000
    classDef fab    fill:#E2EFDA,stroke:#006600,color:#000000
    classDef net    fill:#FFF2CC,stroke:#D6A800,color:#000000
    classDef ace    fill:#FCE4D6,stroke:#C55A11,color:#000000
    classDef plugin fill:#F4CCCC,stroke:#CC0000,color:#000000
    classDef agent  fill:#D9EAD3,stroke:#38761D,color:#000000

    Op([Operator / CI-CD]):::actor

    subgraph BIZ["Business / Intent Plane"]
        Intent["Intent Interface<br/>NL → SID Intent Model"]:::biz
        SLA["SLA Registry<br/>KPI thresholds"]:::biz
        subgraph OSS["OSS / BSS — TMF Open APIs"]
            TMF641["TMF641 Service Order"]:::biz
            TMF654["TMF654 Trouble Ticket"]:::biz
            TMF640["TMF640 Activation & Config"]:::biz
            TMF628["TMF628 Performance Mgmt"]:::biz
            TMF688["TMF688 Event Mgmt"]:::biz
            TMF724["TMF724 AI/ML Model Mgmt"]:::biz
        end
        Intent --> SLA
        Intent --> OSS
    end

    subgraph FABRIC["Agent Fabric Plane — A2A-T / IG1453"]
        Registry["Registry Center<br/>Agent Cards · Auth · Skill Discovery"]:::fab
        Orch["Orchestration Center<br/>Workflow · IG1453A Prompt Meta-Model"]:::fab
        subgraph AGENTS["Domain Agents"]
            RANAgent["RAN Agent<br/>O-RAN xApp / rApp"]:::agent
            CoreAgent["Core NF Agent<br/>AMF / SMF / UPF lifecycle"]:::agent
            SliceAgent["Slice Agent<br/>E2E slice mgmt"]:::agent
            AssurAgent["Assurance Agent<br/>SLA / fault mgmt"]:::agent
            OSSAgent["OSS/BSS Agent<br/>order / ticket handling"]:::agent
        end
        Registry --> Orch
        Orch --> RANAgent
        Orch --> CoreAgent
        Orch --> SliceAgent
        Orch --> AssurAgent
        Orch --> OSSAgent
    end

    subgraph NET["Network Infrastructure Plane"]
        subgraph ORAN["O-RAN"]
            NRRIC["Non-RT RIC — A1 policy"]:::net
            RTRRIC["Near-RT RIC — E2 control"]:::net
            ODU["O-DU / O-RU"]:::net
        end
        subgraph CORE5G["5G Core — 3GPP TS 28.xxx"]
            CoreNFs["AMF / SMF / UPF"]:::net
            NRF["NRF / PCF / NSSF"]:::net
        end
        subgraph TRANS["Transport"]
            Fabric["IP/MPLS / Optical<br/>NETCONF/YANG"]:::net
        end
        NRRIC --> RTRRIC --> ODU
    end

    subgraph ACE["ACE-NET Certification Plane"]
        Core["ACE-NET Core Engine<br/>Lifecycle · Certificate CA"]:::ace
        Policy["Policy Engine<br/>ace-policy.yang"]:::ace
        Evid["Evidence Store<br/>ace-audit.yang"]:::ace
        subgraph TEA["Telecom Environment Adapter"]
            P3GPP["3GPP NBI Plugin"]:::plugin
            PORAN["O-RAN O1/A1/E2 Plugin"]:::plugin
            PNETCONF["NETCONF/YANG Plugin"]:::plugin
            PTMF["TMF OpenAPI Plugin"]:::plugin
            PA2AT["A2A-T Plugin<br/>Agent Card · Task intercept<br/>Fabric fault injection"]:::plugin
            CHAOS["Chaos Injector<br/>network + fabric faults"]:::plugin
        end
        Core --> Policy
        Core --> Evid
        Core --> TEA
    end

    Op -->|Submit AgentProfile<br/>autonomy_level_claim| Core
    Core -->|ComplianceCertificate JWT<br/>layer1 + layer2 + layer3| Op

    BIZ -->|intent → agent tasks via A2A-T| FABRIC
    FABRIC -->|domain agents act on infrastructure| NET

    PA2AT <-.->|intercept registration & tasks| Registry
    PA2AT <-.->|intercept orchestration| Orch
    P3GPP <-.->|3GPP NBI TS 28.530| CoreNFs
    PORAN <-.->|O1 / E2| RTRRIC
    PORAN <-.->|A1| NRRIC
    PNETCONF <-.->|NETCONF/YANG| Fabric
    PTMF <-.->|TMF OpenAPIs| OSS
    CHAOS -.->|Agent Fabric faults<br/>Registry down · task loop<br/>IG1453A malformed prompt| FABRIC
    CHAOS -.->|Network faults<br/>NF crash · link fail · SLA breach| NET
```

## Component Responsibilities

| Component | Responsibility |
|-----------|---------------|
| **A2A-T Plugin** | Validates Agent Cards, acts as peer agent stub, intercepts task lifecycle, injects fabric-level faults, validates IG1453A prompt meta-model |
| **Registry Center** | Agent discovery, authentication, skill management across the Agent Fabric |
| **Orchestration Center** | Multi-agent workflow coordination via IG1453A prompt meta-model |
| **Chaos Injector** | Injects both network-layer and Agent Fabric-layer faults per certification tier |
| **Policy Engine** | Evaluates observed behavior against `ace-policy.yang` rules and ECA definitions |
| **Evidence Store** | Packages audit trail and compliance artifacts as `ace-audit.yang` instances |
