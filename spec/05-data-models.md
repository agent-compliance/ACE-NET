
***

### `spec/05-data-models.md`

```markdown
### 5. Data Models

Data models SHOULD be machine-readable, versioned, and based on open standards like JSON Schema or YANG where applicable.

#### 5.1. Agent Profile

A JSON object describing the agent.

```json
{
  "profileVersion": "1.0",
  "agentId": "urn:uuid:f81d4fae-7dec-11d0-a765-00a0c91e6bf6",
  "provider": "VendorX",
  "agentVersion": "2.1.3",
  "description": "AIOps agent for 5G UPF anomaly detection and remediation.",
  "interfaces": [
    { "type": "3GPP-N35", "version": "16.4.0" },
    { "type": "TMF628", "version": "4.0" }
  ],
  "resourceRequirements": {
    "cpu": "2000m",
    "memory": "4Gi",
    "storage": "10Gi"
  }
}
```

#### 5.2. Compliance Policy Data Model (ace-policy.yang)

For policies to be interoperable and machine-readable, this document proposes a normative data model defined in YANG. The model is designed with modularity and extensibility in mind, drawing inspiration from IETF policy management standards.

```yang
module ace-policy {
  yang-version 1.1;
  namespace "urn:ietf:params:xml:ns:yang:ace-policy";
  prefix acep;

  import ietf-inet-types { prefix inet; }
  import ietf-yang-types { prefix yang; }

  organization "ACE-NET Consortium";
  contact "Editor: <mailto:editor@ace-net.example.com>";
  description
    "This YANG module defines a data model for representing compliance
     policies within the ACE-NET framework. It includes structures for
     policy metadata, scope, a flexible rule engine with various rule
     types, and remediation actions upon violation.";
  revision 2026-06-20 {
    description "Initial version of the ACE-NET compliance policy model.";
    reference "draft-doe-ace-net-framework-00";
  }

  // (The full content of the ace-policy.yang model from the draft is assumed here for brevity)
  // ...
}
```

#### 5.3. Compliance Report and Audit Log Data Model (ace-audit.yang)

To ensure that compliance reports and audit trails are machine-readable, interoperable, and comprehensive, this document proposes a normative YANG data model for their structure. This model provides a formal schema for capturing detailed event-level audit trails and aggregated compliance reporting data.

```yang
module ace-audit {
  yang-version 1.1;
  namespace "urn:ace:net:audit";
  prefix "audit";

  import ietf-yang-types {
    prefix yang;
  }

  description
    "This YANG data model defines the structure for machine-readable
     audit logs and compliance reports within the ACE-NET framework.";

  revision 2024-05-21 {
    description "Initial version.";
    reference "draft-doe-ace-net-framework-00";
  }

  // (The full content of the ace-audit.yang model from the draft is assumed here for brevity)
  // ...
}
```
