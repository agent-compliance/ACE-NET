### 5. Data Models

Data models are machine-readable, versioned, and based on open standards (JSON Schema, YANG). All models MUST be treated as normative unless stated otherwise.

#### 5.1. Agent Profile

The Agent Profile is a signed JSON document submitted by the agent provider to initiate certification. Fields marked REQUIRED must be present; fields marked CONDITIONAL are required when `autonomy_level_claim` is L3 or L4.

```json
{
  "profileVersion": "2.0",
  "agentId": "urn:uuid:f81d4fae-7dec-11d0-a765-00a0c91e6bf6",
  "provider": "VendorX",
  "agentVersion": "3.0.1",
  "description": "Multi-domain complaint handling agent — network, billing, and CRM triage via A2A-T.",
  "autonomy_level_claim": "L4",
  "agentArtifactUrl": "oci://registry.example.com/vendorx/complaint-agent:3.0.1",
  "a2at_endpoint": "https://agent.vendorx.example.com",
  "scenario_profile_ids": ["HVS-5", "HVS-1"],
  "interfaces": [
    { "type": "TMF654", "version": "4.0", "role": "trouble-ticket" },
    { "type": "TMF628", "version": "4.0", "role": "performance-mgmt" },
    { "type": "IG1453", "version": "2.1.0", "role": "a2at-agent-fabric" }
  ],
  "resourceRequirements": {
    "cpu": "2000m",
    "memory": "4Gi",
    "storage": "10Gi"
  },
  "sidecar": {
    "enabled": true,
    "langfuse_project": "ace-net-cert",
    "intercept_tmf_apis": true,
    "intercept_a2at": true
  }
}
```

Field definitions:

| Field | Required | Description |
|-------|----------|-------------|
| `profileVersion` | REQUIRED | Schema version of this Agent Profile document |
| `agentId` | REQUIRED | Stable URN identifying the agent across versions |
| `autonomy_level_claim` | REQUIRED | Declared target autonomy level: L0, L1, L2, L3, or L4 |
| `agentArtifactUrl` | REQUIRED | URI to the deployable container image or Helm chart |
| `a2at_endpoint` | CONDITIONAL | Base URL of the agent's A2A-T service; required for L3/L4 |
| `scenario_profile_ids` | CONDITIONAL | List of HVS identifiers from Section 8; required for L3/L4 |
| `interfaces` | REQUIRED | List of TMF or protocol interfaces the agent consumes |
| `sidecar.intercept_tmf_apis` | RECOMMENDED | Enable TEA TMF OpenAPI Plugin interception for tool-use evidence |
| `sidecar.intercept_a2at` | CONDITIONAL | Enable A2A-T Plugin interception; required for L3/L4 |

The `sidecar` block configures the TEA instrumentation layer. When `intercept_tmf_apis` is true, the TEA TMF OpenAPI Plugin acts as a mock TMF server (equivalent to an MCP server in general agent frameworks), proxying all TMF API calls made by the AUT and logging them for Phase 1 metrics extraction.

#### 5.2. Compliance Policy Data Model (ace-policy.yang)

The `ace-policy.yang` YANG module is the normative schema for compliance policies. Policies are evaluated by the Policy Engine during Layer 1 behavioral certification.

```yang
module ace-policy {
  yang-version 1.1;
  namespace "urn:ace-net:yang:ace-policy";
  prefix acep;

  import ietf-inet-types { prefix inet; }
  import ietf-yang-types { prefix yang; }

  organization "ACE-NET / AgentCert Initiative";
  contact "Editor: <mailto:editor@ace-net.example.com>";
  description
    "YANG module for ACE-NET compliance policies. Defines rules,
     thresholds, ECA definitions, and autonomy-level constraints
     evaluated against AUT behavior during Layer 1 certification.";

  revision 2026-06-29 {
    description "v2.0 — added telecom domain typedefs and A2A-T rule types.";
    reference "draft-doe-ace-net-framework-00";
  }

  typedef autonomy-level {
    type enumeration {
      enum L0 { description "Manual — no autonomous action."; }
      enum L1 { description "Assisted — human approves every action."; }
      enum L2 { description "Partial Self-X — bounded single task."; }
      enum L3 { description "Conditional Self-X — single-domain autonomous."; }
      enum L4 { description "High Self-X — cross-domain intent-driven."; }
    }
    description "TMForum IG1218 autonomy level.";
  }

  typedef cert-tier {
    type uint8 { range "0..4"; }
    description "ACE-NET certification tier (0=interface, 1=behavioral, 2=playbook, 3=a2at, 4=fabric+hvs).";
  }

  typedef hvs-scenario-id {
    type enumeration {
      enum HVS-1 { description "Autonomous Fault Management."; }
      enum HVS-2 { description "Zero-Touch Network Slicing."; }
      enum HVS-3 { description "Self-Optimizing RAN."; }
      enum HVS-4 { description "Proactive SLA Assurance."; }
      enum HVS-5 { description "Complaint Handling (A2A-T reference scenario)."; }
    }
    description "High-Value Scenario identifier per Section 8.";
  }

  typedef tmf-api-id {
    type enumeration {
      enum TMF628; enum TMF639; enum TMF640;
      enum TMF641; enum TMF654; enum TMF688; enum TMF724;
    }
    description "TMForum Open API identifier.";
  }

  container policy {
    leaf policy-id    { type yang:uuid; mandatory true; }
    leaf name         { type string; mandatory true; }
    leaf min-tier     { type cert-tier; default 1; }
    leaf max-tier     { type cert-tier; default 4; }
    leaf behavioral-score-threshold { type decimal64 { fraction-digits 1; }
      default "80.0";
      description "Minimum Layer 1 behavioral score to pass (0.0–100.0)."; }
    leaf behavioral-score-drift-margin { type decimal64 { fraction-digits 1; }
      default "10.0";
      description "Max behavioral score drift before continuous-monitoring re-cert triggers."; }

    list rule {
      key rule-id;
      leaf rule-id    { type yang:uuid; mandatory true; }
      leaf name       { type string; mandatory true; }
      leaf type       { type enumeration {
        enum metric-based-threshold;
        enum state-assertion;
        enum api-call-sequence-validation;
        enum eca-decision-correctness;
        enum a2at-protocol-assertion;
      }; mandatory true; }
      leaf severity   { type enumeration { enum critical; enum major; enum minor; }; default major; }
      leaf description { type string; }
      // Rule parameters are expressed as type-specific augmentations
    }

    list eca-definition {
      key eca-id;
      description "Event-Condition-Action definition for autonomous remediation validation.";
      leaf eca-id     { type yang:uuid; mandatory true; }
      leaf event-type { type string; mandatory true; }
      leaf condition  { type string; description "XPath expression evaluated against network state."; }
      leaf expected-action { type string; description "Expected agent action within declared boundaries."; }
      leaf timeout-seconds { type uint32; default 300; }
    }
  }
}
```

#### 5.3. Compliance Report and Audit Log Data Model (ace-audit.yang)

The `ace-audit.yang` YANG module defines the three-layer evidence structure assembled into the Compliance Certificate by the Evidence Store.

```yang
module ace-audit {
  yang-version 1.1;
  namespace "urn:ace-net:yang:ace-audit";
  prefix audit;

  import ietf-yang-types { prefix yang; }

  description
    "ACE-NET three-layer audit trail and compliance report schema.
     Instances of this model are assembled by the Evidence Store (Phase 3)
     and serialized as the CertificationReport payload.";

  revision 2026-06-29 {
    description "v2.0 — three-layer model aligned to ACE Phase 0-3 pipeline.";
    reference "draft-doe-ace-net-framework-00";
  }

  container certification-report {
    leaf report-id      { type yang:uuid; mandatory true; }
    leaf agent-id       { type string; mandatory true; }
    leaf agent-version  { type string; mandatory true; }
    leaf issued-at      { type yang:date-and-time; mandatory true; }
    leaf issued-by      { type string; default "ACE-NET-CA"; }

    leaf autonomy-level-claimed { type string; mandatory true; }
    leaf autonomy-level-achieved { type string; mandatory true;
      description "Computed by ACE-NET from Layer 3 results. MUST NOT be set by the agent provider."; }

    container layer1 {
      description "Phase 0-2 pipeline output for isolated behavioral certification.";
      leaf behavioral-score      { type decimal64 { fraction-digits 2; } mandatory true; }
      leaf rules-evaluated       { type uint32; mandatory true; }
      leaf rules-passed          { type uint32; mandatory true; }
      leaf tmf-api-conformance   { type enumeration { enum PASS; enum FAIL; } mandatory true; }
      leaf ttd-p95-seconds       { type decimal64 { fraction-digits 2; }
        description "95th percentile Time to Detect across all fault injections."; }
      leaf ttr-p95-seconds       { type decimal64 { fraction-digits 2; }
        description "95th percentile Time to Remediate across all fault injections."; }
      leaf tool-call-accuracy    { type decimal64 { fraction-digits 2; }
        description "Fraction of TMF API calls with correct schema and semantics (0.0–1.0)."; }
    }

    container layer2 {
      description "A2A-T protocol conformance results. Present only when min-tier >= 3.";
      leaf ig1453-version        { type string; }
      leaf agent-card-valid      { type boolean; }
      leaf task-lifecycle        { type enumeration { enum PASS; enum FAIL; enum NOT_RUN; } }
      leaf multi-agent-flow      { type enumeration { enum PASS; enum FAIL; enum NOT_RUN; } }
      leaf error-handling        { type enumeration { enum PASS; enum FAIL; enum NOT_RUN; } }
      leaf prompt-meta-model     { type enumeration { enum PASS; enum FAIL; enum NOT_RUN; } }
      leaf fabric-faults-passed  { type uint8;
        description "Number of fabric fault scenarios passed (max 4 per Layer 2 run)."; }
    }

    list layer3-hvs-result {
      key hvs-id;
      description "Per-HVS scenario results. One entry per scenario in scenario_profile_ids.";
      leaf hvs-id                { type string; mandatory true; }
      leaf status                { type enumeration { enum PASS; enum FAIL; enum NOT_RUN; } mandatory true; }
      leaf elapsed-seconds       { type uint32; }
      leaf sla-seconds           { type uint32;
        description "Declared SLA threshold for this scenario."; }
      leaf intent-fulfilled      { type boolean; }
      leaf failure-reason        { type string;
        description "Present only when status is FAIL."; }
    }

    leaf intent-fulfillment-rate { type decimal64 { fraction-digits 2; }
      description "Fraction of Layer 3 HVS scenarios where intent was fulfilled (0.0–1.0)."; }

    leaf certificate-jwt         { type string;
      description "The signed JWT Compliance Certificate payload (JWS compact serialization)."; }
  }
}
```
