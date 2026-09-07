# ACE-NET: Agent Compliance Engine for Networking

![IETF Draft Status](https://img.shields.io/badge/IETF%20Draft-draft--doe--ace--net--framework--00-blue.svg) ![License](https://img.shields.io/badge/License-IETF%20Trust-lightgrey.svg)

This repository contains the specification, data models, API definitions, and diagrams for the Agent Compliance Engine for Networking (ACE-NET) framework.

---

## 1. Overview

Modern telecommunication networks (5G/6G, AIOps, Edge) are increasingly managed by autonomous or semi-autonomous software "agents." These agents, which range from simple automation scripts to complex AI/ML models and multi-agent systems, are critical for network operations but also introduce significant risks. A misconfigured, faulty, or malicious agent can lead to service degradation, security breaches, or large-scale outages.

ACE-NET is a framework designed to address this challenge by providing a standardized, automated, and auditable system for testing, certifying, and monitoring the compliance of these agents. It adapts the open-source Agent Compliance Engine (ACE) four-phase certification pipeline to the telecom domain, and aligns it with the TM Forum Autonomous Networks body of work — the IG1218 autonomy level taxonomy, the IG1252 evaluation methodology, the IG1256 effectiveness indicators, the IG1401 Level 4 Industry Blueprint (High-Value Scenarios), and the IG1453 Agent-to-Agent Protocol for Telecoms (A2A-T). Together these ensure that agent behavior aligns with operator policies, industry standards, and service level agreements (SLAs) — and that an agent's claimed autonomy level is independently verified rather than self-asserted.

The goal of this project is to provide a blueprint for operators, vendors, and standards bodies to build interoperable systems for agent compliance, fostering a more reliable and secure autonomous networking ecosystem.

---

## 2. The ACE-NET Framework

The framework is built on a few core concepts:

- **Agent Profile:** A signed, machine-readable document describing an agent's identity, capabilities, declared TMF/A2A-T interfaces, `autonomy_level_claim`, and target High-Value Scenarios (HVS).

- **Three-Layer Certification Model:** Certification is assembled from three cumulative evidence layers — **Layer 1 (Behavioral Certification)**, which scores agent behavior under network-layer chaos against an `ace-policy.yang` rule set; **Layer 2 (A2A-T Protocol Certification)**, which validates IG1453 conformance (Agent Card, task lifecycle, error handling, prompt meta-model) for agents claiming L3/L4 autonomy; and **Layer 3 (HVS Scenario Certification)**, which dispatches end-to-end High-Value Scenarios and verifies operator intent fulfillment.

- **Compliance Policy:** A formal, machine-readable set of rules, thresholds, and Event-Condition-Action (ECA) definitions that an agent must follow. Defined using a YANG data model (`ace-policy.yang`).

- **Autonomy Levels and Certification Tiers:** ACE-NET's five certification tiers (Tier 0–4) map directly to the TM Forum IG1218 autonomy levels (L0 Manual through L4 High Self-X), so that an agent's `autonomy_level_achieved` reflects independently-verified evidence rather than a vendor claim.

- **Compliance Certificate:** A verifiable, signed JWT issued to an agent that completes certification, carrying both the `autonomy-level-claimed` and the ACE-NET-computed `autonomy-level-achieved`, plus per-layer evidence (behavioral score, TMF API conformance, A2A-T protocol results, HVS scenario outcomes). This certificate can be used by orchestrators as a gate for deployment and operational permissions.

The architecture is centered around an **ACE-NET Core Engine** that orchestrates the three-layer certification process and a **Telecom Environment Adapter (TEA)** that provides a standardized interface — via six plugin types (3GPP NBI, O-RAN O1/A1/E2, NETCONF/YANG, TMF OpenAPI, A2A-T, and Chaos Injector) — to the diverse systems in a telecom network and its Agent Fabric.

---

## 3. Repository Structure

This repository is organized to separate the narrative specification from the formal data models, API definitions, diagrams, and background reference material.

```
/
├── README.md                   # This file
│
├── spec/                        # Source files for the IETF Internet-Draft
│   ├── 00-front-matter.md
│   ├── 01-introduction.md
│   ├── 02-terminology.md
│   ├── 03-architectural-framework.md
│   ├── 04-operational-flow.md
│   ├── 05-data-models.md
│   ├── 06-management-and-reporting-apis.md
│   ├── 07-deployment-models.md
│   ├── 08-application-to-telecom-environments.md
│   ├── 09-chaos-fault-taxonomy.md
│   ├── 09-integration-with-etsi-zsm.md
│   ├── 10-security-considerations.md
│   ├── 11-iana-considerations.md
│   └── 12-references.md
│
├── models/                      # Formal data models
│   └── yang/
│       ├── ace-policy.yang      # YANG model for Compliance Policies
│       └── ace-audit.yang       # YANG model for the three-layer audit trail / Compliance Report
│
├── api/                          # OpenAPI 3.0 API specifications
│   ├── policy-management.json
│   ├── audit-reporting.json
│   └── agent-orchestrator.json
│
├── diagrams/                     # Architecture and flow diagrams (all Mermaid, .md-native)
│   ├── ace-architecture.md              # Fig. 1 — four-plane reference architecture
│   ├── al-certification-framework.md    # Fig. 2 — autonomy level ↔ certification tier mapping
│   ├── certification-evidence-chain.md  # Fig. 3 — three-layer certification sequence
│   ├── a2at-compliance-flow.md          # Fig. 4 — Layer 2 A2A-T phase sequence
│   ├── chaos-fault-taxonomy.md          # Fig. 5 — network + Agent Fabric fault taxonomy
│   └── hvs-decomposition.md             # High-Value Scenario decomposition
│
└── reference/                    # Background reference material (not versioned as normative spec)
    ├── IG1218F_Autonomous_Networks_Framework_v2.0.0.pdf
    ├── IG1252_Autonomous_Network_Levels_Evaluation_Methodology_v1.2.0.pdf
    ├── IG1256_Autonomous_Network_Effectiveness_Indicators_v3.2.0.pdf
    └── IG1453_Agent_to_Agent_Protocol_for_Telecoms_v2.1.0.pdf
```

| Directory | Description |
|-----------|-------------|
| `/spec` | Contains the full text of the IETF draft, broken down into individual Markdown files for each section. This makes editing and version control more manageable. Note that two files share the "09" prefix (`09-chaos-fault-taxonomy.md` and `09-integration-with-etsi-zsm.md`); the CI build concatenates files in alphanumeric order, so the taxonomy currently renders before the ZSM integration section regardless of their logical section numbers. |
| `/models` | Contains the formal, machine-readable data models. The `yang/` subdirectory holds the ACE-NET YANG modules for compliance policies (`ace-policy.yang`) and the three-layer audit trail (`ace-audit.yang`). |
| `/api` | Contains the OpenAPI 3.0 specifications for the RESTful APIs defined by the framework: policy management, audit/reporting, and agent-orchestrator interaction. |
| `/diagrams` | Contains the diagram sources referenced by the spec — figures 1–5 plus the HVS decomposition. All diagrams are authored as Mermaid, embedded directly in Markdown files, so they render natively on GitHub and in any Markdown viewer without a separate build step. The framework's earlier PlantUML diagrams have been phased out in favor of this `.md`-native format. |
| `/reference` | TM Forum guide documents (IG1218F, IG1252, IG1256, IG1453) used as the normative and informative background for the spec's autonomy-level, evaluation-methodology, effectiveness-indicator, and A2A-T protocol content. Not part of the versioned spec itself. |

---

## 4. Key Artifacts

The primary outputs of this project are the formal specifications and models:

### IETF Internet-Draft

The main specification document, assembled from the files in the `/spec` directory. It defines the four-plane architecture, the three-layer certification model, operational flow, data models, management APIs, deployment models, telecom use cases, the chaos fault taxonomy, ETSI ZSM integration, and security considerations for the ACE-NET framework.

A GitHub Actions workflow (`.github/workflows/build-and-deploy.yml`) concatenates the `/spec` files and publishes the combined draft to GitHub Pages on every push to `main`. Diagrams render inline wherever the Markdown is viewed (GitHub, IDEs, this repo's rendered spec) since all of them are Mermaid.

### YANG Data Models

| Model | Description |
|-------|-------------|
| `models/yang/ace-policy.yang` | Defines compliance policies: rules, ECA definitions, autonomy-level and certification-tier typedefs, HVS scenario identifiers, and TMF API identifiers. |
| `models/yang/ace-audit.yang` | Defines the three-layer audit trail / Compliance Report structure (Layer 1 behavioral results, Layer 2 A2A-T results, Layer 3 HVS results) assembled by the Evidence Store. |

### OpenAPI Specifications

| Specification | Description |
|---------------|-------------|
| `api/policy-management.json` | CRUD API for managing `ace-policy.yang`-based compliance policies. |
| `api/audit-reporting.json` | API for retrieving compliance reports and audit trail records. |
| `api/agent-orchestrator.json` | The critical API for programmatically managing the agent compliance lifecycle — agent submission, test-run status, and Compliance Certificate retrieval — enabling full automation. |

### TM Forum Reference Guides

The `/reference` directory holds the underlying TM Forum guides that ACE-NET's autonomy model, evaluation methodology, and A2A-T protocol certification are built on: **IG1218F** (Autonomous Networks Framework and Autonomy Levels), **IG1252** (Autonomous Network Levels Evaluation Methodology), **IG1256** (Autonomous Network Effectiveness Indicators), and **IG1453** (Agent-to-Agent Protocol for Telecoms).

---

## 5. Contributing

Contributions to the ACE-NET framework are welcome. Whether you are fixing a typo, improving a diagram, clarifying a section of the draft, or proposing an enhancement to a data model, your input is valuable.

Please feel free to:

- **Open an Issue** to report a bug, ask a question, or suggest a new feature.
- **Fork the repository** and submit a Pull Request with your proposed changes.

---

## 6. License

This work is licensed under the terms of the IETF Trust's Legal Provisions Relating to IETF Documents, as described in [BCP 78](https://www.rfc-editor.org/info/bcp78). Please review these documents carefully.
