# 🛡️ SOC Threat Detection Lab

A home Security Operations Center (SOC) lab built to simulate, detect, and triage real-world cyber threats using industry-standard SIEM tooling and the MITRE ATT&CK framework.

---

## 📌 Overview

This project replicates a production SOC environment in a controlled lab setting. It focuses on ingesting logs from multiple sources, detecting adversarial behavior through custom Splunk correlation rules, and mapping findings to MITRE ATT&CK tactics and techniques — the same workflow used by SOC analysts at enterprise organizations.

**15+ threat scenarios simulated and triaged end-to-end.**

---

## 🎯 Objectives

- Build a functional SIEM pipeline from log ingestion to alert triage
- Simulate real attacker behavior (reconnaissance, lateral movement, exfiltration)
- Map detected events to MITRE ATT&CK TTPs
- Practice SOC analyst workflows: alert investigation, escalation decisions, and incident documentation

---

## 🧱 Architecture

```
┌─────────────────────────────────────────────────────────┐
│                      Attack Simulation                   │
│         (Nmap scans, brute force, C2 traffic)           │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│                    Log Sources                           │
│   Linux syslog · auth.log · network traffic (PCAP)      │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│                  Log Forwarding                          │
│               rsyslog → Splunk Forwarder                 │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│               Splunk SIEM (Core)                         │
│   Indexing · Search · Correlation Rules · Dashboards    │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│              Analysis & Response                         │
│   MITRE ATT&CK Mapping · Triage · Incident Reporting   │
└─────────────────────────────────────────────────────────┘
```

---

## 🛠️ Tech Stack

| Component         | Tool / Technology              |
|-------------------|-------------------------------|
| SIEM              | Splunk (Free/Trial)           |
| Log Forwarding    | rsyslog, Splunk Universal Forwarder |
| Traffic Analysis  | Wireshark                     |
| Attack Simulation | Nmap, custom scripts          |
| OS               | Linux (Ubuntu)                |
| Framework        | MITRE ATT&CK                  |

---

## 🔍 Threat Scenarios Covered

| # | Scenario | MITRE Tactic | Technique ID |
|---|----------|-------------|--------------|
| 1 | Port scanning / host discovery | Reconnaissance | T1046 |
| 2 | SSH brute force login attempts | Credential Access | T1110.001 |
| 3 | Failed sudo privilege escalation | Privilege Escalation | T1548.003 |
| 4 | Unauthorized cron job creation | Persistence | T1053.003 |
| 5 | Outbound C2 beacon simulation | Command & Control | T1071.001 |
| 6 | Sensitive file access (passwd, shadow) | Collection | T1005 |
| 7 | Log clearing / tampering | Defense Evasion | T1070.002 |
| 8 | Lateral movement via SSH | Lateral Movement | T1021.004 |

*Additional scenarios documented in `/scenarios/`*

---

## 📊 Splunk Detection Rules (Sample SPL)

**SSH Brute Force Detection**
```spl
index=linux_logs sourcetype=auth
| search "Failed password"
| stats count by src_ip, user
| where count > 5
| eval alert="Potential SSH Brute Force"
```

**Privilege Escalation Attempt**
```spl
index=linux_logs sourcetype=syslog
| search "sudo" AND ("incorrect password" OR "authentication failure")
| stats count by user, host
| where count > 3
```

**Outbound Beacon Detection**
```spl
index=network_logs
| stats count by dest_ip, dest_port
| where count > 50
| eval alert="Possible C2 Beaconing"
```

---

## 📁 Repository Structure

```
soc-threat-detection-lab/
│
├── splunk/
│   ├── searches/          # Saved SPL queries and correlation rules
│   ├── dashboards/        # Exported Splunk dashboard XMLs
│   └── alerts/            # Alert configurations
│
├── scenarios/
│   ├── 01_port_scan/      # Attack script + expected log output
│   ├── 02_ssh_brute/      # Attack script + detection evidence
│   └── ...
│
├── logs/
│   └── sample/            # Sanitized sample log files for reference
│
├── reports/
│   └── incident_reports/  # Incident report templates and examples
│
├── screenshots/           # Dashboard and alert screenshots
│   └── README.md
│
└── README.md
```

---

## 🚀 Getting Started

### Prerequisites
- Linux VM (Ubuntu 20.04+ recommended)
- Splunk Free (download at splunk.com)
- rsyslog (pre-installed on most Ubuntu systems)
- Wireshark

### Setup

**1. Configure rsyslog to forward to Splunk**
```bash
# /etc/rsyslog.conf — add this line
*.* @@127.0.0.1:5514
```

**2. Configure Splunk to receive logs**
```bash
# In Splunk Web: Settings → Data Inputs → UDP/TCP → Add port 5514
# Or via CLI:
$SPLUNK_HOME/bin/splunk add tcp 5514 -sourcetype syslog
```

**3. Restart rsyslog**
```bash
sudo systemctl restart rsyslog
```

**4. Verify logs are flowing in Splunk**
```spl
index=* | head 20
```

---

## 📸 Screenshots

> Screenshots of Splunk dashboards, alert triggers, and MITRE ATT&CK mappings coming soon.
> See `/screenshots/` directory.

---

## 📄 Sample Incident Report

```
INCIDENT REPORT — IR-2024-007
Date: [Date]
Analyst: Harsh Patel
Severity: High

Summary:
  Multiple failed SSH login attempts detected from external IP 192.168.x.x
  targeting user accounts on host ubuntu-lab-01.

MITRE ATT&CK Mapping:
  Tactic:    Credential Access
  Technique: T1110.001 — Brute Force: Password Guessing

Timeline:
  22:14:01 — First failed login attempt detected
  22:14:47 — 10+ failures within 60 seconds — alert triggered
  22:15:00 — Source IP flagged and logged for review

Recommended Action:
  Block source IP via firewall rule
  Enable fail2ban on SSH service
  Review /var/log/auth.log for additional indicators
```

---

## 📚 Key Learnings

- Built end-to-end visibility from raw syslog to actionable SIEM alerts
- Developed intuition for distinguishing noisy/benign events from true positives
- Learned to structure investigations using the MITRE ATT&CK framework
- Practiced writing clear, concise incident reports aligned with SOC workflows

---

## 🔗 Related Projects

- [AI Pentest Assistant](../ai-pentest-assistant) — AI-powered recon and vulnerability triage
- [Security Automation Pipeline](../security-automation-pipeline) — CI/CD security automation
- [Linux & Cloud Hardening](../linux-cloud-hardening) — CIS Benchmark hardening on AWS EC2

---

## 👤 Author

**Harsh Patel**
M.S. Cybersecurity — DePaul University
[LinkedIn](https://linkedin.com/in/harsh-patel-21a14330a) · [GitHub](https://github.com/harsh301002) · [Portfolio](https://hp301002.wixsite.com/mysite)
