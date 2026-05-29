# 📑 Technical Project Report: Standalone Mini-SIEM Deployment & Multi-Stage Incident Response Simulation

[![Wazuh Version](https://img.shields.io/badge/SIEM-Wazuh%20v4.x-blueviolet?style=flat-square)](https://wazuh.com/)
[![Environment](https://img.shields.io/badge/OS-Kali%20Linux-blue?style=flat-square)](https://www.kali.org/)
[![SecOps Validation](https://img.shields.io/badge/Simulation-Nmap%20%7C%20THC--Hydra-red?style=flat-square)](https://github.com/)

---

## 1. Abstract
In the contemporary cybersecurity paradigm, real-time threat telemetry and centralized log analytics are foundational to maintaining an assertive defensive posture. This capstone project details the architectural design, localized deployment, and empirical validation of a standalone Security Information and Event Management (SIEM) cluster engineered via the open-source **Wazuh** ecosystem on a consolidated **Kali Linux** virtual asset.

To systematically evaluate the detection, parsing, and analytical capabilities of the engine, a multi-stage adversary simulation was executed along the early phases of the Cyber Kill Chain. Initial reconnaissance was conducted utilizing `Nmap` for network topology mapping and port discovery, immediately followed by an aggressive, multi-threaded credential-stuffing and online brute-force campaign targeted at the secure shell (`SSH`) daemon via `THC-Hydra`.

The underlying `wazuh-analysisd` engine successfully intercepted raw system log mutations within `/var/log/auth.log`, map-matched the entries through structural XML regex decoders, and generated deterministic, high-severity operational alerts (**Rule IDs 5710 and 5712**). These events were aggregated, correlated, and visualized across multi-dimensional time-series data panels on a customized dashboard. The empirical findings from this research validate that monolithic, localized SIEM deployments provide high-fidelity monitoring, near-zero telemetry lag, and an optimal, low-overhead blueprint for endpoint detection, event correlation, and automated threat tracking.

---

## 2. Introduction
Modern enterprise infrastructure routinely faces a dense volume of sophisticated cyber threats ranging from automated vulnerability hunting to persistent distributed brute-force attacks. Traditional security measures, such as perimeter firewalls or decentralized logging systems, fail to track internal asset mutation once a perimeter is bypassed. Consequently, centralized log parsing, ingestion, and real-time behavioral correlation through SIEM environments have transitioned from optional frameworks to core operational requirements within Modern Security Operations Centers (SOCs).

### 2.1 Problem Statement
While distributed, enterprise-grade SIEM environments (e.g., commercial deployments of Splunk or massive multi-node ELK clusters) provide extensive horizontal scaling, they demand substantial infrastructural resource overhead, introduce restrictive licensing fees, and require intricate network-level configuration barriers. For small-scale enterprises, isolated critical endpoints, and lightweight lab environments, such deployments are prohibitive. 

There is an evident structural need for a compact, optimized, high-efficiency Mini-SIEM solution capable of co-existing within a unified endpoint to monitor system event telemetry locally, parse unstructured log strings into standardized telemetry streams, and generate immediate visual alert clusters during an active host breach.

### 2.2 Project Objectives
* **Architectural Deployment:** Install and maintain a hardened, monolithic instantiation of the Wazuh Manager, Indexer, and Dashboard stacks on a Unix-based target.
* **Telemetry Ingestion Engineering:** Configure underlying local collection directives to actively stream authentication log deltas (`auth.log`) to the log analysis daemon.
* **Multi-Stage Adversarial Simulation:** Model real-world threat actors by implementing automated port discovery scanning (Reconnaissance Phase) and parallelized credential brute-forcing (Exploitation Phase).
* **Validation and Telemetry Correlation:** Verify the accurate processing of security events via rule-matching triggers, visualizing real-time threat spikes on analytical dashboard timelines.

---

## 3. System Architecture & Technical Stack
The structural engineering of this project leverages a monolithic, unified deployment strategy. Both the defensive security monitoring plane and the simulated threat-generation vectors operate on a single host. This configuration maximizes throughput and matches typical localized host-defense systems or endpoint evaluation paradigms.

### 3.1 Architectural Data Flow Diagram

```text
┌─────────────────────────────────────────────────────────────────────────────┐
│                             KALI LINUX HOST LAYER                           │
│                                                                             │
│  ┌───────────────────────┐                  ┌────────────────────────────┐  │
│  │   OFFENSIVE VECTORS   │                  │      DEFENSIVE TARGET      │  │
│  │                       │                  │                            │  │
│  │  [1] Nmap Port Scan   ├─────────────────►│   Local Network Sockets    │  │
│  │  [2] Hydra SSH Attack │                  │   & OpenSSH (Port 22)      │  │
│  └───────────────────────┘                  └─────────────┬──────────────┘  │
│                                                           │                 │
│                                                           ▼ (Mutates)       │
│                                                    /var/log/auth.log        │
│                                                           │                 │
│  ┌────────────────────────────────────────────────────────┼────────────────┐│
│  │ WAZUH CORE ENGINE LAYER                                ▼                ││
│  │                                                 ┌────────────────┐      ││
│  │                                                 │ Log Ingestion  │      ││
│  │                                                 │(wazuh-modulesd)│      ││
│  │                                                 └───────┬────────┘      ││
│  │                                                         │               ││
│  │                                                         ▼ (Raw Stream)  ││
│  │ ┌───────────────────┐                           ┌────────────────┐      ││
│  │ │ Wazuh Dashboard   │◄──────────────────────────┤wazuh-analysisd │      ││
│  │ │ (Visualization UI)│  (Alert Event Indexing)   │(Rules Engine)  │      ││
│  │ └─────────▲─────────┘                           └────────────────┘      ││
│  └───────────┼─────────────────────────────────────────────────────────────┘│
│              │ (Queries via HTTPS)                                          │
│      ┌───────┴──────────────┐                                               │
│      │ Firefox Web Browser  │                                               │
│      └──────────────────────┘                                               │
└─────────────────────────────────────────────────────────────────────────────┘
