# Enterprise-Automated-Desktop-Deployment

## 🎯 Objective
To replace manual, time-consuming workstation setups with a standardized, "Zero-Touch" imaging solution. This project utilizes **Windows Deployment Services (WDS)** and **Microsoft Deployment Toolkit (MDT)** to PXE boot bare-metal clients and automatically deploy a custom Windows 10/11 Enterprise image with drivers and applications pre-configured.

## 🛠 Skills Applied
- **Systems Administration:** Configured WDS to handle PXE (Preboot Execution Environment) requests and DHCP integration.
- **Image Management:** Created "Golden Images" using MDT, managing Operating System .wim files and driver injection.
- **Task Sequence Automation:** Built custom Task Sequences to automate partitioning, domain joining, and local admin creation.
- **Infrastructure Standardization:** Enforced consistent OS configurations across all endpoints to reduce support ticket volume.

## 💻 Technologies
- **Platform:** Windows Server 2022 (Standard)
- **Tools:** Microsoft Deployment Toolkit (MDT), Windows Deployment Services (WDS), Windows ADK (Assessment & Deployment Kit)
- **Client OS:** Windows 10/11 Enterprise

## 📝 Project Workflow

### 1. MDT Workbench Configuration
Established the `DeploymentShare` directory structure and imported the base Windows operating system files. Created a "Standard Client Task Sequence" to define the installation logic (Partition Disk -> Install OS -> Inject Drivers -> Join Domain).

![MDT Configuration](mdt_config.png)

### 2. WDS & Boot Image Integration
Generated a custom `LiteTouchPE_x64.wim` boot image within MDT and imported it into the WDS Boot Images repository. Configured the WDS Responder to listen for PXE requests from unknown clients on the network.

![WDS Configuration](wds_ready.png)

### 3. Execution (PXE Boot & Deployment)
Successfully PXE booted a blank Virtual Machine client. The client received an IP from DHCP, loaded the MDT Boot Image, and executed the Task Sequence to install Windows completely hands-free.

![Deployment Success](pxe_success.png)

## 🔧 Troubleshooting & Challenges (The Real Work)
This project simulated a complex network environment where I encountered and resolved several real-world infrastructure issues:

### 1. DHCP Contention & "Split Scope" Logic
* **Issue:** The client VM was successfully pulling an IP address but failing to locate the WDS Server.
* **Root Cause:** My home network router was responding to DHCP requests faster than my Windows Server, providing an IP address without the necessary PXE boot instructions (Options 66/67).
* **Resolution:** I installed the **DHCP Server Role** on the Windows Server and created a dedicated scope (`192.168.x.200 - .210`). I then authorized the server in AD to prioritize it over the home router for local PXE requests.

### 2. WDS/DHCP Port Conflict & Option 60
* **Issue:** Encountered Error `0xC1040103` when attempting to configure DHCP Option 60 via PowerShell.
* **Resolution:** Diagnosed that the DHCP Service was not yet installed, causing WDS configuration commands to fail. After installing the DHCP role, I configured WDS to **"Listen on DHCP Ports"** and correctly set Option 60 (PXEClient) to ensure the server announced itself as a boot server.

### 3. PXE Timeout & Zero-Touch Automation
* **Issue:** The deployment would abort with "PXE Boot Aborted" because the `Press F12` timeout window was too short for the virtual console latency.
* **Resolution:** Modified the **WDS Boot Policy** for "Unknown Clients" to **"Always continue the PXE boot."** This removed the manual F12 requirement entirely, achieving true "Zero-Touch" automation.

### 4. Firewall Traffic Blocking
* **Issue:** TFTP (Trivial File Transfer Protocol) traffic was being blocked, causing the boot image download to hang.
* **Resolution:** Identified that Windows Server 2022 defaults the Firewall to "On" for Domain networks. I created exception rules for UDP Port 67 (DHCP) and Port 69 (TFTP) to allow the boot traffic to pass.
