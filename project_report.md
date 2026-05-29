# 📑 Technical Project Report: Standalone Mini-SIEM Deployment & Multi-Stage Incident Response Simulation

[![Wazuh Version](https://img.shields.io/badge/SIEM-Wazuh%20v4.x-blueviolet?style=flat-square)](https://wazuh.com/)
[![Environment](https://img.shields.io/badge/OS-Kali%20Linux-blue?style=flat-square)](https://www.kali.org/)
[![SecOps Validation](https://img.shields.io/badge/Simulation-Nmap%20%7C%20THC--Hydra-red?style=flat-square)]()

---

# 1. Abstract

In the contemporary cybersecurity landscape, real-time threat monitoring and centralized log analytics are essential for maintaining a strong defensive posture. This project presents the design, deployment, and validation of a standalone Security Information and Event Management (SIEM) environment using the open-source **Wazuh** platform on a unified **Kali Linux** virtual machine.

To evaluate the effectiveness of the SIEM engine, a multi-stage attack simulation was performed following the early phases of the Cyber Kill Chain. Initial reconnaissance was conducted using `Nmap` for local network and port discovery, followed by an automated SSH brute-force attack using `THC-Hydra`.

The `wazuh-analysisd` engine successfully detected and processed authentication log events from `/var/log/auth.log`, mapped them through Wazuh decoders and rules, and generated high-severity alerts (**Rule IDs 5710 and 5712**). These alerts were indexed and visualized in real time through the Wazuh Dashboard.

The project demonstrates that a localized, lightweight SIEM deployment can provide effective real-time monitoring, event correlation, and threat detection with minimal infrastructure overhead.

---

# 2. Introduction

Modern systems continuously face cyber threats such as automated reconnaissance, unauthorized access attempts, and brute-force attacks. Traditional standalone logging methods and perimeter-based defenses often fail to provide centralized visibility into security events occurring inside a host system.

Security Information and Event Management (SIEM) solutions address this limitation by collecting, analyzing, correlating, and visualizing security telemetry in real time.

## 2.1 Problem Statement

Enterprise-grade SIEM platforms often require large-scale infrastructure, extensive configuration, and expensive licensing models. These requirements make them impractical for:

* Small organizations
* Educational labs
* Lightweight testing environments
* Standalone endpoints

There is a need for a compact and efficient SIEM deployment capable of:

* Collecting local system logs
* Detecting suspicious activity
* Generating alerts
* Visualizing incidents in real time

## 2.2 Project Objectives

The primary objectives of this project are:

* Deploy a standalone Wazuh SIEM environment on Kali Linux
* Configure local authentication log monitoring
* Simulate reconnaissance and brute-force attack scenarios
* Validate real-time alert generation and event correlation
* Visualize detected threats through the Wazuh Dashboard

---

# 3. System Architecture & Technical Stack

The project follows a monolithic deployment model in which both the attack simulation and SIEM monitoring environment operate on the same host machine.

## 3.1 Architectural Data Flow Diagram

```text
┌───────────────────────────────────────────────────────────────────────────┐
│                           KALI LINUX HOST LAYER                           │
│                                                                           │
│  ┌───────────────────────┐        ┌────────────────────────────┐          │
│  │   OFFENSIVE VECTORS   │        │      DEFENSIVE TARGET      │          │
│  │                       │        │                            │          │
│  │ [1] Nmap Port Scan    ├───────►│ Local Network Sockets      │          │
│  │ [2] Hydra SSH Attack  │        │ & OpenSSH (Port 22)        │          │
│  └───────────────────────┘        └─────────────┬──────────────┘          │
│                                                 │                         │
│                                                 ▼ (Mutates)               │
│                                         /var/log/auth.log                 │
│                                                 │                         │
│  ┌──────────────────────────────────────┼───────────────────────────────┐ │
│  │              WAZUH CORE ENGINE LAYER ▼                               │ │
│  │                                                                      │ │
│  │                           ┌────────────────┐                         │ │
│  │                           │ Log Ingestion  │                         │ │
│  │                           │ (wazuh-modulesd)                         │ │
│  │                           └───────┬────────┘                         │ │
│  │                                   │                                  │ │
│  │                                   ▼ (Raw Stream)                     │ │
│  │      ┌───────────────────┐   ┌────────────────┐                      │ │
│  │      │ Wazuh Dashboard   │◄──┤ wazuh-analysisd│                      │ │
│  │      │ (Visualization UI)│   │ (Rules Engine) │                      │ │
│  │      └─────────▲─────────┘   └────────────────┘                      │ │
│  └────────────────┼─────────────────────────────────────────────────────┘ │
│                   │ (Queries via HTTPS)                                   │
│          ┌────────┴─────────────┐                                         │
│          │ Firefox Web Browser  │                                         │
│          └──────────────────────┘                                         │
└───────────────────────────────────────────────────────────────────────────┘
```

---

## 3.2 Component Breakdown

| Component                  | Technology              | Functional Description                                |
| -------------------------- | ----------------------- | ----------------------------------------------------- |
| **Host Operating System**  | Kali Linux (x86_64)     | VirtualBox-based isolated Linux environment           |
| **SIEM Engine**            | `wazuh-manager`         | Handles log collection, decoding, and rule evaluation |
| **Analytics Dashboard**    | Wazuh Dashboard         | Visualizes alerts, timelines, and security metrics    |
| **Target Service**         | OpenSSH (`ssh.service`) | SSH daemon monitored for authentication attempts      |
| **Reconnaissance Tool**    | Nmap v7.x               | Performs local network and port scanning              |
| **Attack Simulation Tool** | THC-Hydra v9.6          | Executes automated SSH brute-force attacks            |

---

# 4. Experimental Methodology & Attack Simulation

The experiment was divided into two stages:

1. Reconnaissance and service discovery
2. Brute-force exploitation simulation

---

## 4.1 Phase 1: Network Reconnaissance

To simulate an attacker identifying available services, an Nmap scan was performed against the local host.

### 4.1.1 Execution Command

```bash
# Scan localhost for open ports
nmap localhost
```

> 📁 Deliverable Asset Reference: `assets/nmap_scan.png`

### Impact

The scan identified active services and generated network connection activity visible in system logs.

---

## 4.2 Phase 2: SSH Brute-Force Simulation

After identifying the SSH service, a brute-force attack simulation was launched using THC-Hydra.

### 4.2.1 Execution Command

```bash
# SSH brute-force attack against localhost
hydra -l root -P /usr/share/john/password.lst ssh://127.0.0.1
```

> 📁 Deliverable Asset References:

* `assets/siem_dashboard.png`
* `assets/siem_threat_alert.png`

### Impact

Hydra generated thousands of failed authentication attempts within a short time interval. These events were captured by the Linux logging subsystem and processed by Wazuh.

The attack generated:

* Multiple failed SSH login events
* High-frequency authentication anomalies
* Alert escalation inside the SIEM dashboard

---

# 5. SIEM Detection & Analysis Workflow

When attack activity modified `/var/log/auth.log`, the Wazuh engine processed the events through multiple analysis stages.

## 5.1 Log Processing Pipeline

```text
[ Raw Log Entry Generation ]
              │
              ▼
[ Decoder Processing ]
              │
              ├──► Identifies service (e.g., sshd)
              ▼
[ Parameter Extraction ]
              │
              ├──► Extracts IP, username, and port
              ▼
[ Rule Evaluation ]
              │
              ├──► Matches security rules
              ▼
[ Alert Generation ]
              │
              └──► Displays alerts on dashboard
```

---

## 5.2 Detection Logic

### Log Collection

The `wazuh-modulesd` component continuously monitors authentication logs and forwards new entries to the analysis engine.

### Decoder Processing

Wazuh decoders parse raw SSH log entries and extract:

* Source IP address
* Username
* Authentication result
* Service information

### Rule Matching

The extracted information is evaluated against predefined Wazuh rules.

#### Important Rule IDs

| Rule ID  | Description                                                                       |
| -------- | --------------------------------------------------------------------------------- |
| **5710** | Detects failed SSH authentication attempts                                        |
| **5712** | Detects multiple repeated authentication failures indicating brute-force activity |

---

# 6. Results & Dashboard Analysis

The Wazuh Dashboard successfully visualized attack telemetry in real time.

## 6.1 Alert Metrics

| Alert Severity         | Approximate Count |
| ---------------------- | ----------------- |
| Medium Severity Alerts | 190+              |
| Low Severity Alerts    | 424               |

---

## 6.2 Hydra Attack Spike

The Threat Hunting dashboard displayed a noticeable spike during the Hydra execution window, confirming successful detection of abnormal authentication activity.

---

## 6.3 Rule Analysis

Rule ID **5712** became one of the dominant alerts during the attack simulation, validating the effectiveness of Wazuh’s brute-force detection capability.

---

# 7. Conclusion & Future Scope

## 7.1 Conclusion

This project demonstrates that a standalone SIEM deployment can effectively provide:

* Real-time threat monitoring
* Log aggregation
* Event correlation
* Automated alert generation

The Wazuh platform successfully detected both reconnaissance and brute-force attack activity within a localized Linux environment.

---

## 7.2 Future Enhancements

Potential improvements for future work include:

### Multi-Endpoint Monitoring

Deploy Wazuh agents on additional Linux and Windows systems for centralized monitoring.

### Automated Active Response

Configure automated firewall blocking using:

* `iptables`
* `nftables`

when brute-force rules are triggered.

### Threat Intelligence Integration

Integrate external threat intelligence feeds such as:

* AlienVault OTX
* MISP

to identify known malicious IP addresses.

---

# 8. References

1. Wazuh Documentation — https://documentation.wazuh.com/
2. Kali Linux Documentation — https://www.kali.org/docs/
3. Nmap Official Documentation — https://nmap.org/book/man.html
4. THC-Hydra GitHub Repository — https://github.com/vanhauser-thc/thc-hydra
