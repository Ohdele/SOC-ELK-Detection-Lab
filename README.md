## PART 1 – SOC Architecture Diagram (ELK Lab)

### Objective
Build a logical SOC architecture diagram using draw.io to model an ELK-based monitoring environment.

---

### Skills
- SOC architecture design
- Network segmentation (lab design)

### Tools
- draw.io

---

### Summary
SOC architecture diagram built in draw.io showing ELK-based monitoring environment.  
Defined private lab network with internal systems, endpoints, and external connectivity via NAT/host adapter.  
Mapped log and communication flows between Windows, Ubuntu, Fleet, Elastic/Kibana, and C2 simulation systems.

<img src="./01_ELK-Diagram/ELK-Stack.png" width="800">


---


# PART 2 – Elasticsearch Deployment

## Objective
Deploy and configure an Elasticsearch instance for centralized log storage.

---

## Skills
- Linux Administration  
- Elasticsearch Installation  
- Systemd Service Management  
- Network Configuration  
- Basic Security Hardening (firewall rules)

## Tools Used
- Ubuntu Server (VM) | Elasticsearch | PowerShell / SSH  

---

## Evidence

**Elasticsearch Service Status (Active Running):**
<img src="./02_ELK-Active/ServerStatus-Active.png" width="800">

---

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

---

## Conclusion
Successfully deployed and validated Elasticsearch as a centralized log storage engine in a controlled lab environment.


---


# PART 3 – Kibana Setup 

## Objective
Set up Kibana to provide a web-based interface for searching and visualizing data stored in Elasticsearch for security monitoring and detection workflows.

---

## Skills
Linux administration  
Network / firewall configuration  

## Tools
Kibana | UFW | Ubuntu Server | SSH  

---

## Steps

### Ref 1: Kibana Service Status (Active)
<img src="03_ELK-Dashboard/1_KibanaActive.png">

Verified Kibana service is running and accessible via browser.


### Ref 2: Kibana Enrollment & Access Setup
<img src="03_ELK-Dashboard/2_ELK-EnrolmentToken.png">

Installed and linked Kibana with Elasticsearch using enrollment token and verification code.

Enabled access through firewall (UFW) to allow browser connection:
`http://192.168.56.10:5601`

Completed authentication using `elastic` credentials and reached Kibana homepage.


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

---

## Skills
- Virtual machine deployment
- Network segmentation awareness
- Windows Server configuration

## Tools

- VirtualBox | Windows Server 2022 | GitHub

---

## Steps
1. Created Windows Server 2022 VM in VirtualBox
2. Configured initial setup and hostname assignment
3. Applied VirtualBox networking (NAT / Host-Only isolation)
4. Verified system boot and console access

<img src="04_ELK-WinServer/WinServer.png">

- Windows Server deployed locally as a controlled target system for SOC log generation and attack simulation.

---

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


---


# PART 5 – Fleet Server for Centralized Agent Management.

## Objective

Deploy Fleet Server and enroll a Windows Server Elastic Agent for centralized log collection and management.

---

## Skills

- Elastic Fleet management
- Agent enrollment and troubleshooting
- TLS/network troubleshooting
- Windows event log ingestion validation

## Tools

- Elastic 9.4.3 | Fleet Server | Elastic Agent | Ubuntu | Windows | Kibana

---

## Steps

### Fleet Server Deployment

- Deployed Fleet Server on Ubuntu ELK instance using HTTPS:
`https://192.168.56.10:8220`

- Generated Fleet Server policy and enrollment token from Kibana Fleet management.

- Installed Elastic Agent Fleet Server and verified successful connection in Kibana.

<img src="05_Fleet-Agents/1-FleetServerConnected.png">


### Windows Elastic Agent Enrollment

- Created Windows agent policy (`DFIR-Windows-policy`) in Fleet.
- Installed Elastic Agent on Windows Server using the Fleet enrollment command.
- Verified network connectivity between Windows Server and Fleet Server.

Enrollment command used:
.\elastic-agent.exe install --url=<Fleet-Server-URL> --enrollment-token=<token> --insecure

- Decision:
Used --insecure to bypass self-signed TLS certificate validation in the lab environment.

---

## Detection Summary

### Investigation Findings:
  - Fleet Server enrollment initially failed due to incorrect package architecture, enrollment issues, and TLS certificate validation errors.
  - Windows Agent enrollment failed with `x509: certificate signed by unknown authority`.
  - Network connectivity and port availability were verified successfully.


### Decision Made:
  - Corrected Fleet Server deployment configuration.
  - Used `--insecure` for the lab environment to bypass self-signed certificate validation.


### Outcome:
  - Fleet Server successfully connected.
  - Windows Elastic Agent enrolled and reported Healthy status.
  - Windows Security logs successfully ingested into Kibana Discover.

<img src="05_Fleet-Agents/2-FleetAgentsHealthy.png">

---

### Windows Log Validation (4672)

- Verified Windows Security events in Kibana Discover.
- Confirmed event collection with Windows Event IDs.

<img src="05_Fleet-Agents/3-WinEventSecurityLogs.png">

### Outcome:
  - Elastic Agent successfully forwarded Windows security telemetry into the Elastic Stack for monitoring and detection workflows.


---


# PART 6 – Sysmon Installation & Event Monitoring

## Objective

Install and configure Sysmon on the Windows Server, then verify successful deployment and event generation for endpoint monitoring and security analysis.

---

## Skills

- Endpoint monitoring
- Windows event analysis
- Sysmon deployment and configuration
- Security telemetry validation


## Tools

- Windows | Sysmon | Olaf | PowerShell | GitHub

---

## Steps

### 1. Sysmon Installation Preparation

- Downloaded Sysmon from Microsoft Sysinternals.
- Extracted Sysmon binaries on the Windows Server.
- Downloaded Olaf's Sysmon configuration file from GitHub.
- Added the Sysmon configuration file to the Sysmon directory.

<img src="06_Sysmon64/1-SysmonConfig.png">


### 2. Sysmon Directory Validation

- Opened PowerShell as Administrator.
- Navigated to the Sysmon installation directory.
- Verified the presence of the Sysmon executable and configuration file.

<img src="06_Sysmon64/2-SysmonDir.png">


### 3. Sysmon Deployment

- Verified the absence of Sysmon service and logs before installation.
- Installed Sysmon using the configured XML file.
- Accepted the Sysmon license agreement to complete deployment.

<img src="06_Sysmon64/3-SysmonInstalled.png">


### 4. Service Verification

- Refreshed Windows Services after installation.
- Confirmed the Sysmon64 service was installed and running.

<img src="06_Sysmon64/4-SysmonSerStatusRunning.png">


### 5. Event Generation Validation

- Opened Windows Event Viewer.
- Navigated to:
  `Applications and Services Logs → Microsoft → Windows → Sysmon → Operational`
- Confirmed Sysmon event generation.
- Validated Event ID 11, which records file creation and overwrite activity.

<img src="06_Sysmon64/5-SysmonEventVWR.png">

---

## Summary

- Investigation Findings:
  - Sysmon was successfully deployed and generating endpoint telemetry.
  - Event ID 11 confirmed visibility into file creation activity.

- Decision Made:
  - Use Sysmon telemetry as an additional endpoint data source for Elastic Stack monitoring.

- Outcome:
  - Windows Server is now generating Sysmon security events, ready for ingestion into the ELK detection pipeline.


---


## Part 7: Sysmon & Microsoft Defender Log Ingestion

### Objective
Configure Elastic Agent to ingest Sysmon and Microsoft Defender event logs from Windows Server into the on-premises Elastic Search instance.

### Skills
- Elastic Agent integration configuration
- Windows Event Log collection
- Sysmon log ingestion
- Microsoft Defender event monitoring
- SIEM data verification and troubleshooting

### Tools
- Kibana | Elastic | Windows | Sysmon | MS Defender | Event Viewer

### Steps

#### Step 1: Configure Sysmon Windows Event Log Integration

- Created a Custom Windows Event Log integration in Elastic.
- Configured:
  - Integration Name: `DFIR-Win-Sysmon`
  - Purpose: Collect Sysmon logs.
- Retrieved the Sysmon Operational channel from Windows Event Viewer:
  - Applications and Services Logs → Microsoft → Windows → Sysmon → Operational
- Added the integration to the existing Windows Agent Policy.
- Saved and deployed the policy changes.

Result:
- DFIR-Win-Sysmon integration successfully added to the Windows Agent Policy.

---

#### Step 2: Configure Microsoft Defender Event Log Integration

- Created a second Custom Windows Event Log integration.
- Configured:
  - Integration Name: `DFIR-Win-Defender`
  - Purpose: Collect Microsoft Defender logs.
- Retrieved the Defender Operational channel from Windows Event Viewer:
  - Applications and Services Logs → Microsoft → Windows → Windows Defender → Operational

Configured monitored Event IDs:

- `1116` - Malware or potentially unwanted software detected.
- `1117` - Action performed to protect the system from malware.
- `5001` - Real-time protection disabled.

Tested Defender monitoring by disabling real-time protection and verifying Event ID 50001 generation.

<img src="./07_SysmonMSDefender/1-EventID5001-Generated.png">

---

#### Step 3: Deploy Microsoft Defender Event Filtering

- Added the Defender channel name into Elastic.
- Configured Advanced Options with required Event IDs:
  - 1116,1117,50001
- Added the integration to the existing Windows Agent Policy.
- Saved and deployed changes.

Result:
- Sysmon and Microsoft Defender logs were configured for ingestion into the on-premises Elastic Search instance.

<img src="./07_SysmonMSDefender/2-Sysmon-MSDefender.png">

---

#### Step 4: Verify Event Log Ingestion

- Opened Discover in Kibana.
- Verified log availability using:
  - `winlog.event_id`
- Confirmed Sysmon and Microsoft Defender event datasets were available.
- Validated Elastic Agent health through Fleet.
- Restarted Elastic Agent when required and refreshed log discovery.

Verification:
- `winlog.event_id: 1` confirmed Sysmon events.
- `winlog.event_id: 5001` confirmed Microsoft Defender events.

---

#### Step 5: Validate Event Provider Data

Verified the `event.provider` field to confirm successful ingestion.

Microsoft Defender verification:

<img src="./07_SysmonMSDefender/3-MSDefenderIngestionVerified.png">

Sysmon verification:

<img src="./07_SysmonMSDefender/4-SysmonIngestionVerified.png">

---

### Summary

- Investigation Findings:
  - Elastic Agent successfully collected Windows Event Logs from Sysmon and Microsoft Defender.
  - Event IDs were filtered to focus on security-relevant activity.
  - Event provider fields confirmed correct log source identification.

- Decision Made:
  - Implement targeted Windows Event Log collection instead of ingesting unnecessary informational events.

- Result / Outcome:
  - Sysmon and Microsoft Defender telemetry are now available in Elasticsearch for security monitoring, detection engineering, and future threat investigation workflows.
