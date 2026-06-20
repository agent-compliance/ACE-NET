### 2. Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 when, and only when, they appear in all capitals, as shown here.

*   **Agent**: Any autonomous or semi-autonomous software component that performs actions or makes decisions within the network environment. Examples include AI/ML models for AIOps, network slice management functions, edge application orchestrators, and automation controllers.
*   **Agent Under Test (AUT)**: An Agent that is currently undergoing compliance testing by the ACE-NET.
*   **Agent Compliance Engine for Networking (ACE-NET)**: The logical system that orchestrates the compliance testing of Agents in a telecom environment.
*   **Compliance Test Suite (CTS)**: A collection of Test Cases designed to verify an Agent's adherence to a specific set of compliance policies.
*   **Agent Profile**: A machine-readable document describing an Agent's identity, capabilities, interfaces, and resource requirements.
*   **Compliance Policy**: A set of formal rules, constraints, and objectives that an Agent must adhere to. These can be derived from technical standards (3GPP), business rules (SLAs), or security regulations. This document recommends a YANG data model for defining these policies.
*   **Telecom Environment Adapter (TEA)**: A crucial ACE-NET component that acts as a middleware layer, translating generic test instructions into specific commands for various network domains and systems (e.g., 5G Core, RAN, OSS/BSS, Cloud Infrastructure).
*   **Network Function (NF)**: A functional block within a network infrastructure, such as the 5G Core's AMF (Access and Mobility Management Function) or UPF (User Plane Function).
*   **Orchestrator**: A system responsible for the management and orchestration of network resources, services, and functions (e.g., ETSI MANO, Kubernetes).
*   **OSS/BSS**: Operations Support Systems and Business Support Systems, which manage the network and customer-facing services, respectively.
*   **Network Slice**: A virtual, end-to-end logical network that includes its own dedicated resources and is optimized for a specific service type (e.g., eMBB, URLLC).
*   **YANG**: A data modeling language used to model configuration data, state data, Remote Procedure Calls (RPCs), and notifications for network management protocols such as NETCONF and RESTCONF.
*   **ETSI ZSM**: The European Telecommunications Standards Institute (ETSI) Zero-touch network and Service Management (ZSM) framework, an architecture designed to enable fully automated network and service management.
