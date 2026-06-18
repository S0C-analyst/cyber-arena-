# CyberArena — SOC Automation Platform (EDR/SIEM + SOAR)

An end-to-end, self-hosted Security Operations Center (SOC) platform combining **Wazuh XDR/SIEM** for detection with a fully automated **SOAR workflow** (n8n + TheHive + AI-powered triage) for orchestration, investigation, and response.

Built as a training/graduation project to gain practical, end-to-end experience across the full security operations lifecycle — from raw detection to automated containment.

## 📋 Overview

| | |
|---|---|
| **Detection** | Wazuh 4.x (Manager, Indexer, Dashboard) — All-in-One on Ubuntu 22.04 LTS |
| **Orchestration (SOAR)** | n8n (self-hosted) |
| **Case Management** | TheHive 4 |
| **Threat Intelligence** | AbuseIPDB, VirusTotal |
| **AI Triage** | Google Gemini |
| **Connectivity** | Tailscale (mesh VPN across distributed components) |
| **Notification** | Email (SMTP) |

## 🏗️ Architecture

```
Wazuh Manager (detects event, level ≥ 10)
        │  webhook (HTTP POST)
        ▼
n8n SOAR Workflow
        ├─► AbuseIPDB        → IP reputation / abuse confidence score
        ├─► VirusTotal       → multi-engine (70+) IP/file reputation
        ├─► Google Gemini    → AI threat assessment & recommended actions
        ├─► TheHive          → case creation, observable, comment, task, status update
        └─► Wazuh API        → authenticate (JWT) → isolate affected agent
        │
        ▼
Email notification to security team

(In parallel, Wazuh's own Active Response auto-blocks malicious IPs
 at the network layer — firewall-drop/iptables — independent of n8n)
```

End-to-end time from detection to case creation: **under 30 seconds**.

## 🛠️ What's Inside

### Part 1 — Wazuh EDR/SIEM Deployment

1. **All-in-One Deployment** — system requirements, pre-installation checklist, automated install via the Wazuh installation assistant, key ports reference.
2. **EDR & Endpoint Security** — agent deployment on Linux and Windows, File Integrity Monitoring (FIM), rootkit detection (rootcheck).
3. **EDR & External Integrations** — enriching alerts with VirusTotal (file hash reputation) and AbuseIPDB (IP reputation) via a custom Python integration script.
4. **Threat Intelligence (CTI)** — automated OSINT feed ingestion (Firehol, Emerging Threats) into CDB lookup lists, with custom detection rules mapped to MITRE ATT&CK.
5. **Vulnerability Management** — Vulnerability Detector against NVD, Canonical, RedHat, and Microsoft feeds with CVSS-based remediation SLAs.
6. **Active Response** — automated IP blocking (`firewall-drop`) on brute-force, SQL injection, and threat-intel rule matches.
7. **Visualization** — SOC dashboards in the Wazuh Dashboard (KQL queries, geo threat maps, alert distribution panels).
8. **Troubleshooting** — diagnostics and resolutions for common real-world issues.

### Part 2 — SOAR Automation Workflow (n8n)

A 13-node n8n workflow that takes over the moment Wazuh fires a high-severity alert:

| Step | Node | Purpose |
|------|------|---------|
| 1 | Webhook Trigger | Receives the Wazuh alert via HTTP POST |
| 2 | Set Variables | Normalizes sourceIP, ruleName, ruleLevel, agentName, agentId, timestamp |
| 3 | AbuseIPDB Lookup | Checks source IP abuse confidence score |
| 4 | VirusTotal Request | Scans the IP against 70+ AV/threat-intel engines |
| 5 | AI Analysis (Gemini) | Generates a threat assessment and recommended actions |
| 6 | TheHive Case | Creates the incident case (tags, severity, TLP/PAP) |
| 7 | Add Observable | Registers the malicious IP as an IOC on the case |
| 8 | Comment | Posts an automated analysis summary to the case |
| 9 | Update Case Status | Sets status to Open, assigns the analyst |
| 10 | Create Task | Adds an "Investigate IP" task for the analyst |
| 11 | Wazuh Auth | Obtains a JWT bearer token from the Wazuh API |
| 12 | Isolate Agent | Triggers active-response network isolation on the affected endpoint |
| 13 | Send Email | Notifies the security team |

Full step-by-step input/output specs, the data-flow map between nodes, manual test procedure (`curl`-based trigger), expected validation results, and a troubleshooting table (webhook errors, API quota limits, Tailscale disconnects, JWT/auth failures) are documented in the SOAR guide.

## 🎯 Key Skills Demonstrated

- Linux system administration & SIEM/XDR deployment
- Detection engineering (custom Wazuh rules, CDB lists, MITRE ATT&CK mapping)
- SOAR design and workflow automation (n8n)
- Case management & incident lifecycle tracking (TheHive)
- Threat intelligence integration (AbuseIPDB, VirusTotal) and OSINT feed automation
- AI-assisted threat triage (Google Gemini)
- Security automation and active response: automated endpoint isolation via SOAR (n8n + Wazuh API), and automated malicious-IP blocking at the network layer (Wazuh `firewall-drop`/iptables)
- Secure distributed connectivity (Tailscale mesh VPN)
- Vulnerability management workflows and CVSS-based prioritization
- SOC dashboarding, KQL querying, and incident troubleshooting

## 📄 Documents

- [`Wazuh_EDR_SIEM_SOC_Guide.docx`](./Wazuh_EDR_SIEM_SOC_Guide.docx) — full EDR/SIEM deployment & operations guide
- [`SOC_Automation_Workflow_Documentation.docx`](./SOC_Automation_Workflow_Documentation.docx) — full SOAR workflow documentation (n8n + TheHive + AI triage)

## ⚠️ Disclaimer

This project was built and tested in an isolated lab environment for educational purposes. API keys, IP addresses, and credentials shown in the documentation are lab-specific or placeholders — replace them with your own before deploying. Always test configuration changes in a non-production environment first.

## 🙋 About

Built as a training/graduation project (CyberArena) to develop practical SOC analyst, detection engineering, and security automation skills end-to-end — from detection (Wazuh) through orchestration and response (n8n + TheHive + AI triage).

Feel free to open an issue or reach out if you have questions or suggestions.
