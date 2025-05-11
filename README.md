# SOC Alert Automation Pipeline

A complete, end-to-end SOC alert triage pipeline that integrates Windows 10 telemetry (via Sysmon) with cloud-hosted SIEM, SOAR, and case management platforms to automate Mimikatz alert detection, enrichment, and escalation.

---

## Tools & Technologies

- **Windows 10 VM** with Sysmon
- **Wazuh** (SIEM)
- **Shuffle** (SOAR automation)
- **TheHive** (Case management)
- **VirusTotal API** (Reputation checking)

---

## Objectives

The goal of this project is to design and implement a fully automated SOC alert triage pipeline that integrates endpoint telemetry (via Sysmon) with a cloud-hosted SIEM (Wazuh), SOAR automation (Shuffle), and case management (TheHive) platforms. The pipeline automates the detection, enrichment, and escalation of Mimikatz alerts in a controlled environment, simulating real-world attack scenarios.

---

## Lab Setup

### 1. Windows 10 VM 

To simulate real-world attacker and endpoint behavior, a dedicated Windows 10 virtual machine (VM) was provisioned using VirtualBox. This endpoint serves as the telemetry source in the lab, designed to generate logs and security events for collection, detection, and triage via Wazuh and TheHive.

**Virtual Machine Settings**

- **RAM: 8GB**
- **Disk Space: 30GB**
- **OS: Windows 10 (64 Bit)**

![image](https://github.com/user-attachments/assets/02727a8e-094e-402e-b278-f144f774b610)


**Sysmon**

**Sysmon** is installed to generate detailed Windows event logs like process creation, network connections, and file changes. In this project, it's used to capture security events when Mimikatz is run, so Wazuh can detect it and automate responses through TheHive and Shuffle.

1. Download Sysmon from https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon
2. Click on the **Download Sysmon** link to download a ZIP file.
3. Extract the contents of the ZIP file to a folder, for example: `C:\Tools\Sysmon`
4. The extracted files should include:
     - `Sysmon.exe` (32-bit)
     - `Sysmon64.exe` (64-bit)
     - License agreement text file

**Sysmon Configuration File**

1. Head to https://github.com/olafhartong/sysmon-modular
2. Open the repository and locate the file `sysmonconfig.xml`
3. Click the file, then click the **Raw** button
4. Right-click anywhere on the page and choose **Save As**
5. Save the file as `sysmonconfig.xml` in your Sysmon folder, e.g., `C:\Downloads\Sysmon`

- **The Sysmon folder should look like this:**

![image](https://github.com/user-attachments/assets/96ec249f-129b-476d-be6a-4aa09b07efa3)

**Running Sysmon**

1. Open Powershell as Administrator
2. In Powershell, navigate to the folder where Sysmon was located
```powershell
   cd "C:\Downloads\Sysmon"
```
3. Run Sysmon with the Configuration File
```powershell
   .\Sysmon64.exe -i sysmonconfig.xml
```
![image](https://github.com/user-attachments/assets/49689296-824d-402a-98a2-85339b9556e5)

4. Verify that Sysmon is Running
```powershell
   Get-Service Sysmon64
```
![image](https://github.com/user-attachments/assets/faf6c6a5-9148-4493-be8b-ecb05b097591)

5. Verify Sysmon logs in Event Viewer

- Go to:
```Settings
   Applications and Services Logs > Microsoft > Windows > Sysmon > Operational
```
![image](https://github.com/user-attachments/assets/69e4bc39-6727-4931-a485-9eb6be50ca22)


### 2. Wazuh and TheHive

In this part of the project, two virtual machines (VMs) were set up in DigitalOcean to host Wazuh (for SIEM) and TheHive (for case management). These VMs are configured to communicate with each other securely with only the necessary ports open through firewall rules.

**Wazuh and TheHive VM Setup**

### 1. Create DigitalOcean Account
- Sign up for a DigitalOcean account (or log in if you already have one).

### 2. Create New Droplets (VMs)
- Go to your DigitalOcean dashboard and click on **Create** → **Droplets**.
- Choose an image (for both VMs, select Ubuntu 20.04 LTS as the base image).
- Select a plan (e.g., Standard, 1GB RAM, 1 vCPU).
- Set up authentication (either SSH keys or password).
- Click on **Create Droplet**.

### 3. Firewall Configuration
Once the VMs are created, navigate to **Networking** in the DigitalOcean dashboard.
- Set up firewall rules to ensure that only necessary ports are accessible for both VMs.
    - **For Wazuh VM**:
      - Allow **SSH (22)** for remote access.
      - Allow **Wazuh ports** (port 1514 for communication with agents).
      - Allow **HTTPS (443)** for any web interfaces.
    - **For TheHive VM**:
      - Allow **SSH (22)** for remote access.
      - Allow **TheHive web interface ports** (default port is 9000 for HTTP).
- Apply the firewall to both VMs.


