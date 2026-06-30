### 12. References

#### Normative References

[RFC2119] Bradner, S., "Key words for use in RFCs to Indicate Requirement Levels", BCP 14, RFC 2119, DOI 10.17487/RFC2119, March 1997.

[RFC8174] Leiba, B., "Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words", BCP 14, RFC 8174, DOI 10.17487/RFC8174, May 2017.

[RFC6241] Enns, R., Bjorklund, M., Schoenwaelder, J., Bierman, A., "Network Configuration Protocol (NETCONF)", RFC 6241, June 2011.

[RFC7950] Bjorklund, M., "The YANG 1.1 Data Modeling Language", RFC 7950, August 2016.

[IG1252] TMForum, "Autonomous Network Levels Evaluation Methodology", IG1252 v1.2.0, <https://www.tmforum.org/resources/introductory-guide/ig1252/>. NORMATIVE: Defines the three-part evaluation approach (effectiveness indicators, technology maturity assessment, resource closed-loop autonomy) and five evaluation dimensions (Execution, Awareness, Analysis, Decision, Intent/Experience) used to measure agent autonomy in ACE-NET certification.

[IG1218F] TMForum, "Autonomous Networks Framework and Autonomy Levels", IG1218F v2.0.0, <https://www.tmforum.org/resources/introductory-guide/ig1218/>. NORMATIVE: Defines the L0–L4 autonomy level classification, KEI/KCI effectiveness indicator hierarchy, Target Architecture components, and the AN Map scenario taxonomy used in ACE-NET certification tiers and Layer 3 HVS selection.

[IG1256] TMForum, "Autonomous Network Effectiveness Indicators", IG1256 v3.2.0, <https://www.tmforum.org/resources/introductory-guide/ig1256/>. NORMATIVE: Defines the Autonomous Network Effectiveness Indicators (ANEIs) reference set — Service Availability, Mean Time to Detect, Mean Time to Repair, Incident Reduction Rate, Automation Ratio, Intelligent Ratio — with measurement formulas and L0–L4 target thresholds used in ACE-NET Layer 1 and Layer 3 evidence evaluation.

[IG1453] TMForum, "Agent-to-Agent Protocol for Telecoms (A2A-T) v2.1.0", IG1453, <https://www.tmforum.org/resources/introductory-guide/ig1453/>. NORMATIVE: The A2A-T protocol specification including Agent Card schema, Registry/Orchestration Center roles, task lifecycle state machine (PENDING → ASSIGNED → RUNNING → COMPLETED/FAILED), error handling contract, and multi-agent collaboration patterns validated in ACE-NET Layer 2 certification.

#### Informative References

[REF-ACE] AgentCert Initiative, "Agent Compliance Engine (ACE) open-source framework", <https://github.com/AgentCert/AgentCert>. The base framework from which ACE-NET is derived. Implements the four-phase certification pipeline (Phase 0–3) on Kubernetes using LitmusChaos, Langfuse, and LiteLLM.

[IG1401] TMForum, "Autonomous Networks Industry Blueprint — Level 4 Target Architecture", IG1401, <https://www.tmforum.org/resources/introductory-guide/ig1401-autonomous-networks-industry-blueprint/>. Defines the High-Value Scenarios (HVS) taxonomy and the five representative scenarios (HVS-1 through HVS-5) used in ACE-NET Layer 3 certification, mapped to the IG1218F AN Map.

[C26.0.910] TMForum Catalyst, "Agent Fabric — A2A-T Runtime Phase III", Catalyst Project C26.0.910. Defines the Agent Fabric runtime (Registry Center, Orchestration Center) used in ACE-NET Layer 2 and Layer 3 certification.

[IG1453A] TMForum, "IG1453A Agent-to-Agent Prompt Meta-Model for Telecoms". Defines the structured prompt schema validated in Layer 2 Phase D of ACE-NET.

[3GPP-28.530] 3GPP, "Management and orchestration; Concepts, use cases and requirements", TS 28.530. Referenced for 5G Core NF lifecycle management interfaces used by the TEA 3GPP NBI Plugin.

[3GPP-28.541] 3GPP, "Management and orchestration; 5G Network Resource Model (NRM); Stage 2 and stage 3", TS 28.541. Referenced for Network Slice NRM used in HVS-2 normative assertions.

[ORAN-WG2] O-RAN Alliance, "Non-RT RIC and A1 Interface", O-RAN.WG2.A1AP-v05.00. Referenced for A1 policy delivery in HVS-3 sub-scenarios.

[ORAN-WG3] O-RAN Alliance, "Near-RT RIC and E2 Interface", O-RAN.WG3.E2SM-v03.00. Referenced for E2 control loop latency requirements in HVS-3.1.

[TMF628] TMForum, "Performance Management API", TMF628, Open API v4.0.

[TMF639] TMForum, "Resource Inventory Management API", TMF639, Open API v4.0.

[TMF640] TMForum, "Activation and Configuration API", TMF640, Open API v4.0.

[TMF641] TMForum, "Service Ordering Management API", TMF641, Open API v4.0.

[TMF654] TMForum, "Trouble Ticket Management API", TMF654, Open API v4.0.

[TMF688] TMForum, "Event Management API", TMF688, Open API v4.0.

[TMF724] TMForum, "AI/ML Model Management API", TMF724, Open API v4.0.

[RFC7519] Jones, M., Bradley, J., Sakimura, N., "JSON Web Token (JWT)", RFC 7519, May 2015. Used for the Compliance Certificate format (Section 4.5).

[RFC3060] Moore, B., "Policy Core Information Model — Version 1 Specification", RFC 3060. Background reference for the ACE-NET policy management approach.

[IETF-ECA] Ma, Q. et al., "A YANG Data Model for ECA-Policy Management", IETF Internet-Draft, <https://datatracker.ietf.org/doc/html/draft-ma-netmod-eca-policy>. Referenced for ECA rule design in `ace-policy.yang`.
