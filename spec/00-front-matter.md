**Network Working Group**
**Internet-Draft**
**Intended status: Informational**
**Expires: December 21, 2026**

**J. Doe**
**AgentCert Initiative**
**June 20, 2026**

### A Framework for Agent Compliance in Autonomous Networking Systems (ACE-NET)
### draft-doe-ace-net-framework-00

### Abstract

The proliferation of autonomous agents, including AI/ML models and complex automation scripts, in modern telecommunication networks (5G/6G, edge computing, AIOps) introduces significant challenges for governance, reliability, and security. These agents, which perform critical functions from network orchestration to real-time traffic management, must operate within strict compliance boundaries. This document proposes the Agent Compliance Engine for Networking (ACE-NET), a specialized framework adapted from the open-source Agent Compliance Engine (ACE). ACE-NET provides a standardized methodology, architecture, data models, and APIs for testing, certifying, and monitoring the compliance of autonomous agents within complex, distributed networking environments. The framework is designed to integrate with existing telecom systems like OSS/BSS, network orchestrators, and 5G/6G core components, as well as management architectures like ETSI ZSM, ensuring that agent behavior aligns with operator policies, industry standards, and service level agreements (SLAs).

### Status of This Memo

This Internet-Draft is submitted in full conformance with the provisions of BCP 78 and BCP 79.

Internet-Drafts are working documents of the Internet Engineering Task Force (IETF). Note that other groups may also distribute working documents as Internet-Drafts. The list of current Internet-Drafts is at https://www.ietf.org/ietf/1id-abstracts.txt.

Internet-Drafts are draft documents valid for a maximum of six months and may be updated, replaced, or obsoleted by other documents at any time. It is inappropriate to use Internet-Drafts as reference material or to cite them other than as "work in progress."

This Internet-Draft will expire on December 21, 2026.

### Copyright Notice

Copyright (c) 2026 IETF Trust and the persons identified as the document authors. All rights reserved.

This document is subject to BCP 78 and the IETF Trust's Legal Provisions Relating to IETF Documents (https://trustee.ietf.org/license-info) in effect on the date of publication of this document. Please review these documents carefully, as they describe your rights and restrictions with respect to this document.

### Table of Contents

1.  [Introduction](#1-introduction)
2.  [Terminology](#2-terminology)
3.  [ACE-NET Architectural Framework](#3-ace-net-architectural-framework)
4.  [Operational Flow](#4-operational-flow)
5.  [Data Models](#5-data-models)
6.  [Management and Reporting APIs](#6-management-and-reporting-apis)
7.  [Deployment Models](#7-deployment-models)
8.  [Application to Telecom Environments (Use Cases)](#8-application-to-telecom-environments-use-cases)
9.  [Integration with ETSI Zero-touch network and Service Management (ZSM)](#9-integration-with-etsi-zero-touch-network-and-service-management-zsm)
10. [Security Considerations](#10-security-considerations)
11. [IANA Considerations](#11-iana-considerations)
12. [References](#12-references)
