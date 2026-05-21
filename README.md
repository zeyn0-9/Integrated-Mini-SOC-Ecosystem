# 🛡️ Integrated Mini-SOC Ecosystem
### Digital Egypt Pioneers Initiative (DEPI) — Round 4

![GitHub repo size](https://img.shields.io/github/repo-size/zeyn0-9/Integrated-Mini-SOC-Ecosystem)
![Last Commit](https://img.shields.io/github/last-commit/zeyn0-9/Integrated-Mini-SOC-Ecosystem)
![License](https://img.shields.io/badge/license-MIT-blue)
![MITRE ATT&CK](https://img.shields.io/badge/MITRE-ATT%26CK-red)
![ELK Stack](https://img.shields.io/badge/SIEM-ELK%20Stack%208.x-yellow)

---

## 📖 Table of Contents
- [Project Overview](#-project-overview)
- [Team](#-team--cai4_iss6_g1)
- [Architecture](#️-architecture)
- [Technology Stack](#️-technology-stack)
- [Detection Coverage](#-detection-coverage-mitre-attck)
- [ELK Deployment](#-elk-stack-deployment-on-ubuntu-vps)
- [Use Cases](#-use-cases--simulations)
- [Key Configuration](#️-key-configuration)
- [Troubleshooting](#-troubleshooting)
- [Results](#-results)

---

## 👥 Team — CAI4_ISS6_G1

| Name | Role |
|------|------|
| Adham Hany Mohamed | Infrastructure & SIEM Lead |
| Ahmed Mokhtar Helmy | Detection Engineering |
| Ibrahim Hussein Ibrahim | Endpoint Security & EDR |
| Zeynep Ahmed Abd El Hamied | Incident Response & Documentation |

**Track:** Infrastructure & Security (Information Security Analyst)

---

## 📌 Project Overview

The **Integrated Mini-SOC Ecosystem** is a fully functional, production-grade Security Operations Center built entirely on open-source and cloud-based technologies. It simulates real-world enterprise SOC operations, including:

- ✅ Centralized log collection from Windows, Linux, and network sources
- ✅ Real-time threat detection mapped to MITRE ATT&CK
- ✅ Endpoint Detection & Response (EDR) via Elastic Defend
- ✅ Active Directory security monitoring
- ✅ Incident response workflows and playbooks

---

## 🏗️ Architecture

<img width="900" alt="Mini SOC Flow Architecture" src="https://github.com/user-attachments/assets/0a98b985-005d-4d54-ba86-f68f3602600e" />

---

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
| T1190 | Exploit Public-Facing Application | WAF + Application Logs |
| T1110.004 | Brute Force: Credential Stuffing | SSH Logs |

---

## 🚀 ELK Stack Deployment on Ubuntu VPS

> Deployed on **Contabo VPS** — Ubuntu 22.04 LTS | 12 GB RAM | 200 GB Storage

### 📋 Prerequisites

```bash
sudo apt update && sudo apt upgrade -y
sudo sysctl -w vm.max_map_count=262144
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
```

---

### 1️⃣ Install Elasticsearch

```bash
# Add GPG Key
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | \
  sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg

# Add Repository
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] \
  https://artifacts.elastic.co/packages/8.x/apt stable main" | \
  sudo tee /etc/apt/sources.list.d/elastic-8.x.list

# Install
sudo apt update && sudo apt install elasticsearch -y
```

<img width="477" height="108" alt="Elasticsearch Installation" src="https://github.com/user-attachments/assets/c1d7f70f-4ed5-422a-ad55-80d04e3093ab" />

### JVM Heap Tuning (Critical for 12 GB RAM)

```bash
sudo nano /etc/elasticsearch/jvm.options.d/memory.options
```

```
-Xms4g
-Xmx4g
```

### Start & Enable

```bash
sudo systemctl enable elasticsearch --now
sudo systemctl status elasticsearch
```

<img width="624" height="186" alt="Elasticsearch Running" src="https://github.com/user-attachments/assets/18ec29f0-8be6-448e-806f-6862b7e8c523" />

### Reset Admin Password

```bash
sudo /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic
```

### Test Connection

```bash
curl -u elastic -k https://localhost:9200
```

---

### 2️⃣ Install Kibana

```bash
sudo apt install kibana -y
```

<img width="624" height="288" alt="Kibana Installation" src="https://github.com/user-attachments/assets/f0cf7513-b136-4fb7-852c-45559640927c" />

```bash
# Generate Enrollment Token
sudo /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana
```

<img width="624" height="44" alt="Enrollment Token" src="https://github.com/user-attachments/assets/5c81feba-0e58-4b63-83bf-a3d3077e9612" />

```bash
# Start & Enable
sudo systemctl enable kibana --now
```

<img width="624" height="40" alt="Kibana Enable" src="https://github.com/user-attachments/assets/9d8e0ca1-cc1d-43d9-8cd3-69570b8a254a" />

**Access Dashboard:**
```
https://YOUR_VPS_IP:5601
Username: elastic
Password: (generated above)
```

---

### 3️⃣ SSL/TLS Certificate Generation

```bash
# Generate Certificate Authority (CA)
sudo /usr/share/elasticsearch/bin/elasticsearch-certutil ca --pem
```

<img width="624" height="231" alt="CA Generation" src="https://github.com/user-attachments/assets/9941e6cd-f6d4-4017-a21a-56ff54d16d9e" />

<img width="624" height="240" alt="CA Generation 2" src="https://github.com/user-attachments/assets/4418868f-bd71-4dd7-b296-c8b1022a8aa0" />

<img width="624" height="84" alt="CA Generation 3" src="https://github.com/user-attachments/assets/3f720a22-6aca-42ce-a836-ca3390a75964" />

```bash
# Generate Fleet Server Certificate
sudo /usr/share/elasticsearch/bin/elasticsearch-certutil cert \
  --name fleet-server \
  --ca-cert ca/ca.crt \
  --ca-key ca/ca.key \
  --pem \
  --dns YOUR_VPS_IP \
  --ip YOUR_VPS_IP

# Copy certificates
sudo mkdir -p /etc/kibana/certs
sudo cp fleet-server.crt fleet-server.key ca/ca.crt /etc/kibana/certs/
```

---

### 4️⃣ Fleet Server Setup

```bash
# Download Elastic Agent
curl -L -O https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-8.19.15-linux-x86_64.tar.gz
tar xzvf elastic-agent-8.19.15-linux-x86_64.tar.gz
cd elastic-agent-8.19.15-linux-x86_64
```

<img width="624" height="123" alt="Download Agent" src="https://github.com/user-attachments/assets/4cf2ca49-53e5-4a87-bd1a-210551beceb3" />

```bash
# Enroll Fleet Server
sudo elastic-agent enroll \
  --url=https://YOUR_VPS_IP:8220 \
  --fleet-server-es=https://localhost:9200 \
  --fleet-server-es-ca-trusted-fingerprint=YOUR_FINGERPRINT \
  --fleet-server-service-token=YOUR_TOKEN \
  --fleet-server-policy=fleet-server-policy \
  --fleet-server-es-insecure \
  --insecure \
  --fleet-server-timeout=10m
```

<img width="624" height="195" alt="Fleet Enrollment" src="https://github.com/user-attachments/assets/aac52c39-f28a-4b34-8d22-10b9ba9a79bb" />

<img width="506" height="345" alt="Fleet Server Running" src="https://github.com/user-attachments/assets/b6cf2a2f-f8ae-4d40-8f42-4444e398ed3f" />

---

### 5️⃣ Install Elastic Agent on Endpoints

#### Windows

```powershell
$ProgressPreference = 'SilentlyContinue'
Invoke-WebRequest -Uri https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-8.19.15-windows-x86_64.zip `
  -OutFile elastic-agent-8.19.15-windows-x86_64.zip
Expand-Archive .\elastic-agent-8.19.15-windows-x86_64.zip -DestinationPath .
cd elastic-agent-8.19.15-windows-x86_64
.\elastic-agent.exe install `
  --url=https://YOUR_VPS_IP:8220 `
  --enrollment-token=YOUR_TOKEN
```

<img width="664" height="222" alt="Windows Agent Install" src="https://github.com/user-attachments/assets/b87ef82c-97b2-412c-bd02-5c2fcbd15620" />

#### Linux (Ubuntu)

```bash
curl -L -O https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-8.19.15-linux-x86_64.tar.gz
tar xzvf elastic-agent-8.19.15-linux-x86_64.tar.gz
cd elastic-agent-8.19.15-linux-x86_64
sudo ./elastic-agent install \
  --url=https://YOUR_VPS_IP:8220 \
  --enrollment-token=YOUR_TOKEN
```

<img width="672" height="145" alt="Linux Agent Install" src="https://github.com/user-attachments/assets/8fcfe5a4-3ad6-4922-b0cc-efa9b212c804" />

---

### 6️⃣ Verify Healthy Enrollment

```
Kibana → Management → Fleet → Agents
```

<img width="636" height="279" alt="Fleet Agents Healthy" src="https://github.com/user-attachments/assets/8d68f79d-20ed-48f1-8ace-12737b7856e2" />

All agents should display a **Healthy** status ✅

---

## ⚙️ Key Configuration

### JVM Heap Tuning (Elasticsearch on 12 GB RAM)

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

<img width="896" height="311" alt="Fleet Server & Windows VM + Ubuntu VM are Healthy" src="https://github.com/user-attachments/assets/cd748966-121c-47a0bd0398f01a9e90e1" />

---

## 🚨 Use Cases & Simulations

### 1. Brute Force Detection

```
Attack:  hydra / failed login attempts → Event ID 4625
Alert:   5+ failed logins from the same IP within 1 minute
MITRE:   T1110
```

<img width="700" alt="Failed Logon on Windows Server" src="https://github.com/user-attachments/assets/b99e4365-d882-4950-90d7-f05dcada8bfe" />

---

### 2. SSH Brute Force on VPS ✅ (Real Attack Detected)

```
Attack:  Real SSH brute force from IP: 102.47.78.221
Alert:   High severity — 2 alerts triggered
Host:    Contabo VPS
Date:    May 18, 2026
MITRE:   T1110.004 (Brute Force: Credential Stuffing)
```

<img width="846" height="336" alt="SSH Brute Force Alert" src="https://github.com/user-attachments/assets/dba60bec-5d26-479d-b274-d0b63d5d217c" />

---

### 3. SQL Injection Attempt ✅ (Real Attack Detected)

```
Attack:  SQL Injection attempt on OWASP Juice Shop
Alert:   High severity — 2 alerts triggered
Host:    ubuntu-dmz
MITRE:   T1190 (Exploit Public-Facing Application)
```

<img width="831" height="350" alt="SQL Injection Alert" src="https://github.com/user-attachments/assets/fdf7f1dd-18f3-46bd-8a2e-ba2209b5f8e6" />

---

## 🔧 Troubleshooting

### Access Denied — service_tokens

```bash
sudo chown elasticsearch:elasticsearch /etc/elasticsearch/service_tokens
sudo chmod 660 /etc/elasticsearch/service_tokens
sudo systemctl restart elasticsearch
```

### Elasticsearch Not Starting

```bash
# Check logs
sudo journalctl -u elasticsearch -f

# Verify JVM heap
cat /etc/elasticsearch/jvm.options.d/memory.options

# Check memory
free -h
```

### Agent Not Connecting to Fleet

```bash
sudo elastic-agent unenroll
sudo elastic-agent enroll \
  --url=https://VPS_IP:8220 \
  --enrollment-token=NEW_TOKEN \
  --insecure
sudo systemctl restart elastic-agent
```

---

## 📁 Repository Structure

```
Integrated-Mini-SOC-Ecosystem/
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

- ✅ **4 agents** enrolled and healthy (DC + Win11 + Ubuntu + Fleet Server)
- ✅ **Real-time log ingestion** from Windows Event Logs, AD, and Linux syslog
- ✅ **Custom detection rules** mapped to MITRE ATT&CK
- ✅ **Elastic Defend EDR** active on all endpoints
- ✅ **TLS-secured** communication between all agents and Fleet Server
- ✅ **Real attacks detected** — SSH Brute Force & SQL Injection

---

## 📚 References

- [Elastic Documentation](https://www.elastic.co/docs)
- [MITRE ATT&CK Framework](https://attack.mitre.org)
- [Elastic Agent Installation](https://www.elastic.co/guide/en/fleet/current/install-fleet-managed-elastic-agent.html)
- [Fleet Server Setup](https://www.elastic.co/guide/en/fleet/current/fleet-server.html)

---

> **Note:** This project was developed for educational purposes as part of the DEPI program.
> All configurations use lab/test credentials and should not be used in production as-is.
