
### 6. Management and Reporting APIs

To facilitate the programmatic interaction required in automated network environments, ACE-NET defines a set of RESTful APIs. These APIs allow for the management of compliance policies, the retrieval of audit and reporting data, and the orchestration of test runs. The APIs SHOULD be implemented following OpenAPI 3.0 specifications.

#### 6.1. Policy Management API

This API provides standard CRUD (Create, Read, Update, Delete) operations for compliance policies. It allows operators and automation systems to manage the lifecycle of policies stored in the ACE-NET repository. The request and response bodies for this API are directly derived from the `ace-policy.yang` model defined in Section 5.2.

*(Note: The OpenAPI 3.0 specification snippet from the draft is assumed here for brevity.)*

#### 6.2. Audit and Reporting API

This API provides endpoints for retrieving compliance reports and detailed audit trail records. It is the primary interface for external systems (e.g., dashboards, SIEMs, orchestrators) to consume the results of ACE-NET's compliance checks. The data structures served by this API are defined by the `ace-audit.yang` model in Section 5.3.

*(Note: The OpenAPI 3.0 specification snippet from the draft is assumed here for brevity.)*

#### 6.3. Agent-Orchestrator Interaction API

This API is used by network orchestrators and CI/CD systems to programmatically manage the compliance lifecycle of an Agent Under Test (AUT). It provides endpoints to submit an agent for testing, query the status of the test run, and retrieve the final compliance certificate.

The following OpenAPI 3.0 specification defines this interface.

```json
{
  "openapi": "3.0.0",
  "info": {
    "title": "ACE-NET Agent-Orchestrator Interaction API",
    "version": "1.0.0",
    "description": "API for network orchestrators to manage the compliance lifecycle of an Agent Under Test (AUT) within the ACE-NET framework."
  },
  "servers": [
    {
      "url": "https://ace-net.example.com/api/v1"
    }
  ],
  "paths": {
    "/agents/submissions": {
      "post": {
        "summary": "Submit Agent for Testing",
        "description": "Submits an Agent Profile to initiate a new compliance test run. The test is asynchronous. A unique testRunId is returned to track the execution.",
        "operationId": "submitAgentForTesting",
        "requestBody": {
          "required": true,
          "content": {
            "application/json": {
              "schema": {
                "$ref": "#/components/schemas/AgentProfile"
              }
            }
          }
        },
        "responses": {
          "202": {
            "description": "Accepted. The test run has been successfully initiated.",
            "content": {
              "application/json": {
                "schema": {
                  "type": "object",
                  "properties": {
                    "testRunId": {
                      "type": "string",
                      "format": "uuid",
                      "description": "The unique identifier for this test run.",
                      "example": "a7b1cde2-f345-6789-abcd-ef0123456789"
                    }
                  }
                }
              }
            }
          },
          "400": { "description": "Bad Request. The provided agent profile is invalid." }
        }
      }
    },
    "/test-runs/{testRunId}": {
      "get": {
        "summary": "Query Test Status",
        "description": "Retrieves the current status of a compliance test run using its testRunId.",
        "operationId": "getTestRunStatus",
        "parameters": [
          { "name": "testRunId", "in": "path", "required": true, "schema": { "type": "string", "format": "uuid" } }
        ],
        "responses": {
          "200": {
            "description": "Current status of the test run.",
            "content": { "application/json": { "schema": { "$ref": "#/components/schemas/TestRunStatus" } } }
          },
          "404": { "description": "Not Found. The specified testRunId does not exist." }
        }
      }
    },
    "/test-runs/{testRunId}/certificate": {
      "get": {
        "summary": "Retrieve Compliance Certificate",
        "description": "Fetches the final compliance result for a completed test run. If the test was successful, this includes a machine-readable Compliance Certificate.",
        "operationId": "getComplianceCertificate",
        "parameters": [
          { "name": "testRunId", "in": "path", "required": true, "schema": { "type": "string", "format": "uuid" } }
        ],
        "responses": {
          "200": {
            "description": "Compliance result and certificate.",
            "content": { "application/json": { "schema": { "$ref": "#/components/schemas/ComplianceResult" } } }
          },
          "404": { "description": "Not Found. The specified testRunId does not exist." },
          "422": { "description": "Unprocessable Entity. The test run has not yet completed." }
        }
      }
    }
  },
  "components": {
    "schemas": {
      "AgentProfile": {
        "type": "object",
        "properties": {
          "agentName": { "type": "string", "example": "5G-UPF-Agent" },
          "agentVersion": { "type": "string", "example": "v2.1.4" },
          "vendor": { "type": "string", "example": "NetCo" },
          "agentArtifactUrl": { "type": "string", "format": "uri", "description": "URL to the agent's deployment artifact (e.g., container image, package).", "example": "oci://registry.example.com/netco/upf-agent:v2.1.4" },
          "policyProfileId": { "type": "string", "description": "ID of the compliance policy profile to use for testing.", "example": "nist-security-baseline-v3" }
        },
        "required": ["agentName", "agentVersion", "vendor", "agentArtifactUrl", "policyProfileId"]
      },
      "TestRunStatus": {
        "type": "object",
        "properties": {
          "testRunId": { "type": "string", "format": "uuid" },
          "status": { "type": "string", "enum": ["PENDING", "PROVISIONING", "RUNNING", "COMPLETED", "FAILED"] },
          "lastUpdated": { "type": "string", "format": "date-time" }
        }
      },
      "ComplianceResult": {
        "type": "object",
        "properties": {
          "testRunId": { "type": "string", "format": "uuid" },
          "overallStatus": { "type": "string", "enum": ["SUCCESS", "FAILURE"], "description": "The final pass/fail outcome of the test run." },
          "summaryReportUrl": { "type": "string", "format": "uri", "description": "A URL to a human-readable summary of the test results." },
          "complianceCertificate": { "type": "string", "description": "A signed JSON Web Token (JWT) issued by ACE-NET upon successful compliance verification. The JWT contains claims about the agent and the test context. Present only if overallStatus is 'SUCCESS'." }
        }
      }
    }
  }
}
```
