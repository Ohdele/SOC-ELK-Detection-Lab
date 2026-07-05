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
- Ubuntu Server (VM)  
- Elasticsearch  
- PowerShell / SSH  

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

## Next Phase (Part 3)
Set up Kibana to provide a web-based interface for searching and visualizing data stored in Elasticsearch.
