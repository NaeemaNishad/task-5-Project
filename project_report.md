# 📑 Technical Project Report: Standalone Mini-SIEM Deployment & Multi-Stage Incident Response Simulation

[![Wazuh Version](https://img.shields.io/badge/SIEM-Wazuh%20v4.x-blueviolet?style=flat-square)](https://wazuh.com/)
[![Environment](https://img.shields.io/badge/OS-Kali%20Linux-blue?style=flat-square)](https://www.kali.org/)
[![SecOps Validation](https://img.shields.io/badge/Simulation-Nmap%20%7C%20THC--Hydra-red?style=flat-square)]()

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
To address the issues highlighted above, this project achieves the following milestone objectives:
* **Architectural Deployment:** Install and maintain a hardened, monolithic instantiation of the Wazuh Manager, Indexer, and Dashboard stacks on a Unix-based target.
* **Telemetry Ingestion Engineering:** Configure underlying local collection directives to actively stream authentication log deltas (`auth.log`) to the log analysis daemon.
* **Multi-Stage Adversarial Simulation:** Model real-world threat actors by implementing automated port discovery scanning (Reconnaissance Phase) and parallelized credential brute-forcing (Exploitation Phase).
* **Validation and Telemetry Correlation:** Verify the accurate processing of security events via rule-matching triggers, visualizing real-time threat spikes on analytical dashboard timelines.

---

## 3. System Architecture & Technical Stack
The structural engineering of this project leverages a monolithic, unified deployment strategy. Both the defensive security monitoring plane and the simulated threat-generation vectors operate on a single host. This configuration maximizes throughput and matches typical localized host-defense systems or endpoint evaluation paradigms.

### 3.1 Architectural Data Flow Diagram

┌─────────────────────────────────────────────────────────────────────────────┐
│                            KALI LINUX HOST LAYER                            │
│                                                                             │
│  ┌───────────────────────┐                  ┌────────────────────────────┐  │
│  │   OFFENSIVE VECTORS   │                  │      DEFENSIVE TARGET      │  │
│  │                       │                  │                            │  │
│  │  [1] Nmap Port Scan   ├─────────────────►│   Local Network Sockets    │  │
│  │  [2] Hydra SSH Attack │                  │   & OpenSSH (Port 22)      │  │
│  └───────────────────────┘                  └─────────────┬──────────────┘  │
│                                                           │                 │
│                                                           ▼ (Mutates)       │
│                                                   /var/log/auth.log         │
│                                                           │                 │
│  ┌────────────────────────────────────────────────────────┼────────────────┐│
│  │ WAZUH CORE ENGINE LAYER                                ▼                ││
│  │                                                ┌────────────────┐       ││
│  │                                                │ Log Ingestion  │       ││
│  │                                                │ (wazuh-modulesd)       ││
│  │                                                └───────┬────────┘       ││
│  │                                                        │                ││
│  │                                                        ▼ (Raw Stream)   ││
│  │ ┌───────────────────┐                          ┌────────────────┐       ││
│  │ │ Wazuh Dashboard   │◄─────────────────────────┤wazuh-analysisd │       ││
│  │ │ (Visualization UI)│  (Alert Event Indexing)  │(Rules Engine)  │       ││
│  │ └─────────▲─────────┘                          └────────────────┘       ││
│  └───────────┼─────────────────────────────────────────────────────────────┘│
│              │ (Queries via HTTPS)                                          │
│      ┌───────┴──────────────┐                                               │
│      │ Firefox Web Browser  │                                               │
│      └──────────────────────┘                                               │
└─────────────────────────────────────────────────────────────────────────────┘

### 3.2 Comprehensive Component Breakdown

| Component | Technology | Functional Description |
| :--- | :--- | :--- |
| **Host Operating System** | Kali Linux (x86_64 Architecture) | Deployed within a hardware-isolated VirtualBox hypervisor instance with allocated resource optimization. |
| **Wazuh Core SIEM Engine** | `wazuh-manager` | Executed as a system background daemon. Houses log monitoring agents, XML pattern decoders, and the rule-matching evaluation block (`wazuh-analysisd`). |
| **Wazuh Analytics Suite** | Wazuh Dashboard | Integrated OpenSearch-derived graphical visualization layer executing an internal web node to parse index mappings into graphs and timelines. |
| **Target Security Service** | `ssh.service` (OpenSSH) | Standard SSH server exposed locally on TCP Port 22, configured to record authorization milestones into the Linux syslog system. |
| **Network Mapping Engine** | Nmap v7.9x | Open-source network exploration utility used to synthesize rapid connection handshakes against local interfaces. |
| **Parallel Login Auditor** | THC-Hydra v9.6 | Multi-threaded network login crack engine utilized to execute high-velocity parallel authentication sequences using credential matrices. |

## 4. Experimental Methodology & Offensive Simulation
The evaluation of the security engineering stack was broken down into two distinct operational execution tracks, matching the lifecycle of standard cyber attacks.

### 4.1 Phase 1: Network Reconnaissance and Footprinting
To simulate an early-stage threat actor scouting for attack surfaces, an aggressive local port discovery scan was directed against the loopback network topology.

#### 4.1.1 Execution Command

# Target loopback interface to map open structural services
nmap localhost

> 📁 **Deliverable Asset Reference:** `assets/nmap_scan.png`

* **Infrastructural Impact:** The engine systematically evaluated common TCP ports to determine socket availability. This network reconnaissance activity was simulated and successfully logged, leaving clear telemetry trails of local connection requests within the kernel audit logs.

---

### 4.2 Phase 2: Online Brute-Force Exploitation
Following target profiling, an active, high-velocity brute-force authentication campaign was launched targeting the root administrative profile across the OpenSSH daemon path.

#### 4.2.1 Execution Command
```bash
# Trigger highly parallelized SSH login crack loop against localhost
hydra -l root -P /usr/share/john/password.lst ssh://127.0.0.1

> 📁 **Deliverable Asset Reference:** `assets/siem_dashboard.png` & `assets/siem_threat_alert.png`

* **Infrastructural Impact:** Hydra established maximum parallel child tasks to push hundreds of credential pairs from the system wordlist against the local host. This structural volume overwhelmed the SSH standard authentication limits, leading to connection drops. However, it successfully triggered **3,559 distinct authentication errors** inside the Linux auditing layer within a tight 35-second window.

---

## 5. SIEM Detection and Analysis Logic
When the offensive vector generated raw log noise inside `/var/log/auth.log`, the defensive layer processed the telemetry stream through distinct structural parsing phases.

### 5.1 Step-by-Step Log Processing Flow

[ Raw Log Entry Generation ]
             │
             ▼
[ Decoder Regex Processing ]  ──► Identifies application string (e.g., "sshd")
             │
             ▼
[ Parameter Extraction ]     ──► Isolates Source IP, Username, Target Port
             │
             ▼
[ Rule Engine Evaluation ]    ──► Matches extracted criteria against ruleset
             │
             ▼
[ Alert Indexing & Display ]  ──► Populates Dashboards (Rule IDs 5710 / 5712)

### Ingestion & Processing Pipeline

* **Ingestion Loop:** The `wazuh-modulesd` service constantly polls the active local log files, tracking line additions as raw, unformatted text strings.
* **Decoder Normalization:** The log engine matches the raw text against standard SSH decoders. It extracts variables like source address (`127.0.0.1`), targeted system account name (`root`), and connection type.
* **Rule Comparison & Escalation:** The extracted key-value data is run through the Wazuh rules logic tree to flag anomalous events:
  * **Rule ID 5710:** Triggers upon catching a single standalone instance of an invalid username/password authentication attempt.
  * **Rule ID 5712:** Triggers when a high-frequency threshold of sequential rule 5710 matches occurs within a brief window, identifying an active, automated multi-threaded brute-force intrusion vector.

---

## 6. Project Results and Dashboard Metrics

The validation of the project was confirmed through the live analytics dashboard. Upon opening the **Threat Hunting** and **Security Events** visualization engines, the metrics confirmed total structural capture of the simulation events.

### Telemetry Volumetrics
As referenced via `assets/siem_dashboard_live.png`, the system monitored hundreds of separate log variations, tracking the following metrics inside the localized host environment:

| Alert Severity | Total Count |
| :--- | :--- |
| **Medium Severity Alerts** | 190+ |
| **Low Severity Alerts** | 424 |

### The Hydra Spike
In the Threat Hunting Dashboard layout (`assets/siem_dashboard.png`), the time-series line graph displayed a distinct vertical bar spike tracking right at the execution time of the Hydra command. This provides clear visual confirmation of anomalous behavior over baseline operations.

### Rule Dominance Analysis
**Rule ID 5712** emerged as a dominant alert within the top metrics cards on the dashboard, showing that the system accurately distinguished routine configuration checks from a dedicated, automated attack attempt.

---

## 7. Conclusion and Future Scope

### 7.1 Key Learnings
This project successfully demonstrates that an enterprise-grade SIEM solution can be condensed into a monolithic, localized deployment framework without sacrificing real-time threat detection accuracy. Through the integration of automated log parsing and rules-based signature comparison, the Wazuh environment successfully flagged both network scouting (Nmap) and high-velocity brute-force exploit attempts (Hydra). This provides definitive, auditable confirmation of complex incidents on a single interface.

### 7.2 Future Scope & System Enhancements
To build on this foundation for production environments, the architecture can be extended through three primary engineering paths:

* **Multi-Node Endpoint Expansion:** Transition the current standalone model into a distributed environment by deploying lightweight, remote agents across separate Windows Server and Ubuntu endpoints to aggregate logs over secure TLS tunnels.
* **Automated Active Response Escalation:** Configure the internal `<active-response>` engine within `ossec.conf` to automatically run system firewall drop blocks (`iptables`/`nftables`) against source IPs that trigger **Rule 5712**, isolating attackers instantly.
* **Advanced Threat Intelligence Ingestion:** Integrate commercial cyber threat intelligence feeds (e.g., AlienVault OTX, MISP) to map incoming connection requests against known malicious IP address databases.
