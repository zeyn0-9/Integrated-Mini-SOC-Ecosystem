# 🛡️ Integrated Mini-SOC Ecosystem
### Digital Egypt Pioneers Initiative (DEPI) — Round 4

![GitHub repo size](https://img.shields.io/github/repo-size/zeyn0-9/Integrated-Mini--Soc)
![Last Commit](https://img.shields.io/github/last-commit/zeyn0-9/Integrated-Mini--Soc)
![License](https://img.shields.io/badge/license-MIT-blue)
![MITRE ATT&CK](https://img.shields.io/badge/MITRE-ATT%26CK-red)
![ELK Stack](https://img.shields.io/badge/SIEM-ELK%20Stack%208.x-yellow)

---

## 👥 Team — CAI4_ISS6_G1
| Name | Role |
|------|------|
| Adham Hany Mohamed | Infrastructure & SIEM Lead |
| Ahmed Mokhtar Helmy | Detection Engineering |
| Ibrahim Hussien Ibrahim | Endpoint Security & EDR |
| Zeynep Ahmed Apd El-Hamied | Incident Response & Documentation |

**Track:** Infrastructure & Security (Information Security Analyst)

---

## 📌 Project Overview

The **Integrated Mini-SOC Ecosystem** is a fully functional, production-grade Security Operations Center built entirely on open-source and cloud-based technologies. It simulates real-world enterprise SOC operations including:

- ✅ Centralized log collection from Windows, Linux, and network sources
- ✅ Real-time threat detection mapped to MITRE ATT&CK
- ✅ Endpoint Detection & Response (EDR) via Elastic Defend
- ✅ Active Directory security monitoring
- ✅ Incident response workflows and playbooks

---

## 🏗️ Architecture

<img width="900"  alt="Mini SOC Flow Architecture" src="https://github.com/user-attachments/assets/0a98b985-005d-4d54-ba86-f68f3602600e" />


## 🛠️ Technology Stack

| Component | Technology | Version |
|-----------|-----------|---------|
| **SIEM** | ELK Stack (Elasticsearch + Logstash + Kibana) | 8.x |
| **EDR** | Elastic Defend | 8.19.15 |
| **Agent Management** | Fleet Server | 8.19.15 |
| **Domain Controller** | Windows Server 2022 AD | soc.local |
| **Workstation** | Windows 11 Pro | Domain-joined |
| **Linux Endpoint** | Ubuntu Desktop | 22.04 |
| **Cloud Platform** | Contabo VPS | Ubuntu 22.04 |
| **Detection Framework** | MITRE ATT&CK | v14 |

---

## 🎯 Detection Coverage (MITRE ATT&CK)

| Technique ID | Technique Name | Data Source |
|-------------|----------------|-------------|
| T1110 | Brute Force | Windows Security Logs (4625) |
| T1059.001 | PowerShell Execution | Sysmon + Elastic Defend |
| T1136.002 | Create Domain Account | AD Logs (4720) |
| T1078 | Valid Accounts | Windows Logon Events (4624) |
| T1482 | Domain Trust Discovery | AD Logs (4769) |
| T1021 | Remote Services | Network + SMB Logs |
| T1078.002 | Domain Accounts | Security Event Logs |
| T1098 | Account Manipulation | AD Logs (4728, 4732) |

---

## ⚙️ Key Configuration

### JVM Heap Tuning (Elasticsearch on 12GB RAM)
```bash
# /etc/elasticsearch/jvm.options.d/memory.options
-Xms4g
-Xmx4g
```

### Fleet Server Enrollment
```bash
sudo elastic-agent enroll \
  --url=https://VPS_IP:8220 \
  --fleet-server-es=https://localhost:9200 \
  --fleet-server-es-ca-trusted-fingerprint=YOUR_FINGERPRINT \
  --fleet-server-service-token=YOUR_TOKEN \
  --fleet-server-policy=fleet-server-policy \
  --fleet-server-timeout=10m
```

### Elastic Agent — Linux Endpoint
```bash
curl -L -O https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-8.19.15-linux-x86_64.tar.gz
tar xzvf elastic-agent-8.19.15-linux-x86_64.tar.gz
cd elastic-agent-8.19.15-linux-x86_64
sudo ./elastic-agent install \
  --url=https://VPS_IP:8220 \
  --enrollment-token=YOUR_TOKEN
```

### Elastic Agent — Windows Endpoint
```powershell
.\elastic-agent.exe install `
  --url=https://VPS_IP:8220 `
  --enrollment-token=YOUR_TOKEN
```
<img width="896" height="311" alt="Fleet Server & windows Vm + Ubuntu Vm are Healty" src="https://github.com/user-attachments/assets/cd748966-121c-47a0-bd03-98f01a9e90e1" />

---

## 🚨 Use Cases & Simulations

### 1. Brute Force Detection

Attack:  hydra / failed login attempts → Event ID 4625
Alert:   5+ failed logins from same IP in 1 minute
MITRE:   T1110

<img width="700"  alt="Failed Logon on Windows Server" src="https://github.com/user-attachments/assets/b99e4365-d882-4950-90d7-f05dcada8bfe" />

---
### 2. SSH Brute Force on VPS ✅ (Real Attack Detected)
Attack:  Real SSH brute force from IP: 102.47.78.221
Alert:   High severity — 2 alerts triggered
Host:    Contabo VPS
Date:    May 18, 2026
MITRE:   T1110.004 (Brute Force: Credential Stuffing)

<img width="846" height="336" alt="SSH Brute Force Alert" src="https://github.com/user-attachments/assets/dba60bec-5d26-479d-b274-d0b63d5d217c" />

---
### 3. SQL Injection Attempt

Attack:  SQL Injection Attempt on OWASP Juice Shop
Alert:   High severity — 2 alerts triggered
Host:    ubuntu-dmz
MITRE:   T1190 (Exploit Public-Facing Application)
 <img width="831" height="350" alt="SQL Injection Alert" src="https://github.com/user-attachments/assets/fdf7f1dd-18f3-46bd-8a2e-ba2209b5f8e6" />


---

## 🔧 Troubleshooting

### Access Denied — service_tokens
```bash
sudo chown elasticsearch:elasticsearch /etc/elasticsearch/service_tokens
sudo chmod 660 /etc/elasticsearch/service_tokens
```

### Elasticsearch not starting
```bash
# Check logs
sudo journalctl -u elasticsearch -f

# Verify JVM heap
cat /etc/elasticsearch/jvm.options.d/memory.options
```

### Agent not connecting to Fleet
```bash
# Re-enroll agent
sudo elastic-agent unenroll
sudo elastic-agent enroll --url=https://VPS_IP:8220 \
  --enrollment-token=NEW_TOKEN --insecure
```

---

## 📁 Repository Structure

```
Integrated-Mini--Soc/
│
├── README.md
├── docs/
│   └── DEPI_Final_Report.docx
├── configs/
│   ├── elasticsearch/
│   │   └── jvm.options
│   └── fleet/
│       └── enrollment-guide.md
├── detection-rules/
│   ├── brute-force.md
│   ├── powershell-execution.md
│   └── new-admin-user.md
├── playbooks/
│   └── incident-response-playbook.md
└── screenshots/
    ├── architecture.png
    ├── fleet-agents-healthy.png
    └── kibana-dashboard.png
```

---

## 📊 Results

- ✅ **4 Agents** enrolled and Healthy (DC + Win11 + Ubuntu + Fleet Server)
- ✅ **Real-time log ingestion** from Windows Event Logs, AD, Linux syslog
- ✅ **Custom detection rules** mapped to MITRE ATT&CK
- ✅ **Elastic Defend EDR** active on all endpoints
- ✅ **TLS-secured** communication between all agents and Fleet Server

---

## 📚 References

- [Elastic Documentation](https://www.elastic.co/docs)
- [MITRE ATT&CK Framework](https://attack.mitre.org)
- [Elastic Agent Installation](https://www.elastic.co/guide/en/fleet/current/install-fleet-managed-elastic-agent.html)
- [Fleet Server Setup](https://www.elastic.co/guide/en/fleet/current/fleet-server.html)

---

> **Note:** This project was developed for educational purposes as part of the DEPI program.
> All configurations use lab/test credentials and should not be used in production as-is.
