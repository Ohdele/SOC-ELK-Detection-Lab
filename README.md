## PART 1 – SOC Architecture Diagram (ELK Lab)

### Objective
Build a logical SOC architecture diagram using draw.io to model an ELK-based monitoring environment.

### Skills
- SOC architecture design
- Network segmentation (lab design)

### Tools
- draw.io

### Summary
SOC architecture diagram built in draw.io showing ELK-based monitoring environment.  
Defined private lab network with internal systems, endpoints, and external connectivity via NAT/host adapter.  
Mapped log and communication flows between Windows, Ubuntu, Fleet, Elastic/Kibana, and C2 simulation systems.

<img src="./01_ELK-Diagram/ELK-Stack.png" width="800">


---


# PART 2 – Elasticsearch Deployment

## Objective
Deploy and configure an Elasticsearch instance for centralized log storage.

## Skills
- Linux Administration  
- Elasticsearch Installation  
- Systemd Service Management  
- Network Configuration  
- Basic Security Hardening (firewall rules)

## Tools Used
- Ubuntu Server (VM) | Elasticsearch | PowerShell / SSH  

## Evidence

**Elasticsearch Service Status (Active Running):**
<img src="./02_ELK-Active/ServerStatus-Active.png" width="800">

## Implementation Steps

### 1. Lab Network Setup
- Configured VirtualBox NAT + Host-Only network for isolated lab communication.
- Ensured all VMs were attached to the same Host-Only subnet for internal connectivity.

### 2. System Access & Preparation
- Connected to Ubuntu server via SSH from analyst/host machine.
- Verified connectivity using: `ip a`
- Updated system packages:
  - `sudo apt-get update && sudo apt-get upgrade -y`

### 3. Elasticsearch Installation
- Selected Linux package: **Deb (x86_64)**
- Downloaded package via terminal using `wget <link>`
- Confirmed package using: `ls`
- Installed Elasticsearch:
  - `sudo dpkg -i elasticsearch*.deb`
- Captured and stored auto-generated security credentials for authentication.

### 4. Network Configuration
- Edited Elasticsearch configuration:
  - `sudo nano /etc/elasticsearch/elasticsearch.yml`
- Set:
  - `network.host` → Host-Only IP
  - `http.port` → `9200` (default confirmed)
- Applied access control via Host-Only networking (lab-isolated access model)

### 5. Service Management (systemd)
- systemctl daemon-reload
- systemctl enable elasticsearch
- systemctl start elasticsearch
- Verify status: active (running)

## Conclusion
Successfully deployed and validated Elasticsearch as a centralized log storage engine in a controlled lab environment.


---


# PART 3 – Kibana Setup 

## Objective
Set up Kibana to provide a web-based interface for searching and visualizing data stored in Elasticsearch for security monitoring and detection workflows.

## Skills
Linux administration  
Network / firewall configuration  

## Tools
Kibana | UFW | Ubuntu Server | SSH  

## Steps

### Ref 1: Kibana Service Status (Active)
<img src="03_ELK-Dashboard/1_KibanaActive.png">

Verified Kibana service is running and accessible via browser.

---

### Ref 2: Kibana Enrollment & Access Setup
<img src="03_ELK-Dashboard/2_ELK-EnrolmentToken.png">

Installed and linked Kibana with Elasticsearch using enrollment token and verification code.

Enabled access through firewall (UFW) to allow browser connection:
`http://192.168.56.10:5601`

Completed authentication using `elastic` credentials and reached Kibana homepage.

---

### Ref 3: Kibana Alerts / Security Interface
<img src="03_ELK-Dashboard/3_KibanaAlerts.png">

Validated Kibana UI access and security/detection features functionality.

---

## Conclusion
Kibana successfully installed, configured, and connected to Elasticsearch.


---


# PART 4 – Windows Server Target VM

## Objective
Deploy a local Windows Server VM as a target endpoint for log generation and SOC detection.

## Skills
- Virtual machine deployment
- Network segmentation awareness
- Windows Server configuration

## Tools
- VirtualBox
- Windows Server 2022
- GitHub

## Steps
1. Created Windows Server 2022 VM in VirtualBox
2. Configured initial setup and hostname assignment
3. Applied VirtualBox networking (NAT / Host-Only isolation)
4. Verified system boot and console access

<img src="04_ELK-WinServer/WinServer.png">

- Windows Server deployed locally as a controlled target system for SOC log generation and attack simulation.

## Summary

### Investigation Findings
- Cloud-style VPC segmentation does not apply in VirtualBox.
- Isolation is achieved through VirtualBox NAT/Host-Only networking.
- Segmentation limits the attack blast radius and reduces opportunities for lateral movement.

### Decision Made
- Kept Windows Server isolated from the ELK stack unless intentionally placed on the same internal network.
- Maintained Windows Firewall default configuration and relied on VirtualBox networking for lab isolation.

### Outcome
- Windows Server successfully deployed as an isolated target VM.
- Lab architecture reflects the security principle of network segmentation and blast-radius reduction.
- Environment is ready for log generation, attack simulation, and ELK log ingestion.
