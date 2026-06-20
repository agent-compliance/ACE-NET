### 4. Operational Flow

The ACE-NET process can be broken down into distinct phases, from initial agent registration to continuous monitoring.

#### 4.1. Phase 1: Onboarding and Registration

An Agent provider submits their Agent to the operator. This process includes providing a cryptographically signed **Agent Profile** that details the Agent's identity, purpose, and technical specifications. This profile is ingested into the ACE-NET's Policy & Profile Repository, often via the Agent-Orchestrator Interaction API (see Section 6.3).

#### 4.2. Phase 2: Test Selection and Configuration

The ACE-NET Core Engine analyzes the submitted Agent Profile. Based on the Agent's declared function (e.g., "AIOps Root Cause Analysis for 5G Core") and the operator's active Compliance Policies (defined in YANG), the engine selects the relevant Compliance Test Suites from the repository. For example, it might select suites for 5G Core API interaction, resource consumption limits, and data access policies.

#### 4.3. Phase 3: Test Execution

This is the active testing phase, orchestrated by the ACE-NET Core:
1.  **Environment Provisioning**: ACE-NET instructs the Orchestrator (via the TEA) to provision an isolated test environment. This could be a dedicated lab, a cloud sandbox, or a "digital twin" of a portion of the production network.
2.  **Agent Deployment**: The Agent Under Test (AUT) is deployed into this isolated environment.
3.  **Test Execution**: The Core Engine executes the selected Test Cases sequentially or in parallel.
    *   A test case involves the TEA sending a **stimulus** to the AUT or the environment.
    *   The TEA then **monitors** the AUT's direct responses and its indirect actions on the simulated network environment, checking for violations against the rules defined in the active policies.
    *   All interactions, logs, and state changes are captured for analysis.
4.  **Environment Teardown**: Once testing is complete, the environment is securely torn down.

The following sequence diagram illustrates the interactions during this phase.

```plantuml
@startuml
!theme vibrant

title Phase 3: Test Execution Sequence

actor Operator
participant "ACE-NET Core" as Core
participant "Network Orchestrator" as Orch
participant "Telecom Env. Adapter\n(TEA)" as TEA
box "Test Environment" #LightBlue
  participant "Test Stimulus\n& Monitor" as Stim
  participant "Agent Under Test\n(AUT)" as AUT
end box


== Test Environment Provisioning ==
Core -> Orch: ProvisionTestEnv(testSpec)
activate Orch
Orch -> TEA: CreateSandbox(testSpec)
activate TEA
TEA -> Stim: setup_environment()
activate Stim
Stim --> TEA: ack
deactivate Stim
TEA --> Orch: ack
deactivate TEA
Orch --> Core: ack
deactivate Orch

== Agent Deployment ==
Core -> TEA: DeployAgent(agentArtifactUrl)
activate TEA
TEA -> Stim: deploy_agent(agentArtifactUrl)
activate Stim
Stim -> AUT: <<create>>
Stim -> AUT: configure()
activate AUT
AUT --> Stim: ready
deactivate AUT
Stim --> TEA: ack(agentEndpoint)
deactivate Stim
TEA --> Core: ack
deactivate TEA

== Test Stimulus & Monitoring ==
Core -> TEA: ExecuteTest(policyProfileId)
activate TEA
TEA -> Stim: start_monitoring()
activate Stim
TEA -> Stim: apply_stimulus(stimulus_data)
Stim -> AUT: <<stimulus>>
activate AUT
AUT --> Stim: (logs, metrics, traffic)
deactivate AUT
Stim --> TEA: (streaming data)
deactivate Stim
TEA -> Core: (streaming data)
note right of TEA: Data is forwarded for real-time analysis against policy.

== Environment Teardown ==
Core -> Orch: TeardownTestEnv(testRunId)
activate Orch
Orch -> TEA: DestroySandbox(testRunId)
activate TEA
TEA -> Stim: teardown()
activate Stim
Stim -> AUT: <<destroy>>
Stim --> TEA: ack
deactivate Stim
TEA --> Orch: ack
deactivate TEA
Orch --> Core: ack
deactivate Orch

@enduml

Figure 2: Test Execution Sequence Diagram

4.4. Phase 4: Reporting and Certification
The Log & Report Generator processes the collected data. It compares the observed behavior against the expected outcomes defined in the test cases and policies.

A detailed Compliance Report is generated, structured according to the ace-audit.yang model (see Section 5.3), which can be retrieved via the Reporting API (see Section 6.2).

A machine-readable Compliance Certificate (e.g., a signed JSON Web Token - JWT) is issued if the Agent passes all required tests. This certificate can be programmatically verified by the Orchestrator before allowing the Agent to be deployed or granted permissions in the production network. If a violation occurs, the remediation actions defined in the policy may be triggered.

4.5. Phase 5: Continuous In-Life Compliance
Compliance is not a one-time event. ACE-NET can be used for continuous monitoring of deployed agents. This can be achieved by:

Periodically re-running a subset of non-disruptive tests against production agents.

Using the TEA to monitor agent behavior and API interactions in real-time, flagging any deviations from the policies defined in their original certification. This can leverage Event-Condition-Action (ECA) rules defined in the policy model.