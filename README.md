# 📑 Standalone Mini-SIEM Deployment & Incident Response Simulation

[![Wazuh Version](https://img.shields.io/badge/SIEM-Wazuh%20v4.x-blueviolet?style=flat-square)](https://wazuh.com/)
[![Environment](https://img.shields.io/badge/OS-Kali%20Linux-blue?style=flat-square)](https://www.kali.org/)
[![SecOps Validation](https://img.shields.io/badge/Simulation-Nmap%20%7C%20THC--Hydra-red?style=flat-square)]()

A compact, optimized, high-efficiency Mini-SIEM solution engineered using the open-source **Wazuh** ecosystem on a consolidated **Kali Linux** host. This repository showcases the engineering required to monitor localized endpoint logs, parse unstructured telemetry, and detect active multi-stage adversary simulations in real time.

📖 **Looking for the deep-dive technical breakdown?** Read the full [PROJECT_REPORT.md](PROJECT_REPORT.md).

---

## ⚡ Quick Project Overview

* **Defensive Stack:** Hardened monolithic installation of the Wazuh Manager, Indexer, and Dashboard engine processing system security anomalies (`/var/log/auth.log`).
* **Offensive Simulation:** Emulated initial adversarial footprinting using `Nmap` network port discovery and high-velocity parallel credential brute-forcing via `THC-Hydra`.
* **Detection Mechanism:** Structural XML regex rule decoders (`wazuh-analysisd`) map-matching log deltas to flag anomalies under **Rule IDs 5710 & 5712** with direct mapping to the **MITRE ATT&CK Framework (T1110)**.

---

## 📁 Repository Directory Structure

```text
task-5:project/
├── assets/
│   ├── nmap_scan.png           # Execution telemetry for port discovery simulation
│   ├── siem_dashboard.png      # Time-series alert spike inside Wazuh UI
│   └── siem_threat_alert.png   # Rule 5712 high-severity cluster metric panel
├── config/
│   └── ossec.conf              # Local file monitoring directive snippets
├── logs/
│   └── sample_auth.log         # Raw log footprint generated during simulation
├── rules/
│   └── custom_ssh_rules.xml    # Automated parsing logic & MITRE mapping schemas
├── PROJECT_REPORT.md           # Deep-dive technical project report (Large Document)
└── README.md                   # Repository landing page & brief overview
