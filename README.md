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

VirtualBox is used as the hypervisor to run the Windows VM locally.

- [VirtualBox Download](https://www.virtualbox.org/wiki/Downloads)
- VirtualBox Version: `7.x`
- Extension Pack: Installed (for additional features like clipboard sharing and USB passthrough)

To simulate real-world attacker and endpoint behavior, a dedicated Windows 10 virtual machine (VM) was created. This endpoint serves as the telemetry source in the lab, designed to generate logs and security events for collection, detection, and triage via Wazuh and TheHive.

**Virtual Machine Settings**

| Setting        | Value             |
|----------------|------------------|
| VM Name        | `WIN10-ENDPOINT` |
| OS Type        | Windows 10 (64-bit) |
| Memory         | 8 GB              |
| vCPUs          | 2                 |
| Disk Size      | 50 GB (Dynamic)   |
| Network        | NAT + Host-Only   |


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

### Wazuh and TheHive VM Setup

**DigitalOcean Account**
- Sign up for a DigitalOcean account (or log in if you already have one).

**Create New Droplets (VMs)**
1. Go to your DigitalOcean dashboard and click on **Create** → **Droplets**.
2. Choose an image (for both VMs, select Ubuntu 20.04 LTS as the base image).
3. Select a plan (e.g., Standard, 1GB RAM, 1 vCPU).
4. Set up authentication (either SSH keys or password).
5. Click on **Create Droplet**.

![image](https://github.com/user-attachments/assets/7b1aec42-5728-4b3e-bb04-52a4cf40f3f1)


**Firewall Configuration**
Once the VMs are created, navigate to **Networking** in the DigitalOcean dashboard and set up firewall rules to ensure that only necessary ports are accessible for both VMs.

**Inbound Rules:**
Allow only your IP address to access all necessary ports over both TCP and UDP protocols. This restricts unauthorized access while enabling full control from your machine.

- **Protocol**: TCP & UDP  
- **Port Range**: All  
- **Source**: Your IP address (e.g., your home or office public IP)

![image](https://github.com/user-attachments/assets/42e39223-e9ca-43e4-87ea-465141d51374)


**Outbound Rules:**
Permit full outbound access to enable the VMs to reach external services.

- **ICMP**: All destinations (for ping and diagnostics)  
- **TCP/UDP**: All ports to all IPv4 and IPv6 destinations

![image](https://github.com/user-attachments/assets/cb3cd518-f739-46d4-8419-6d5b571666e3)

### Wazuh 4.7 Installation

1. Use SSH to connect to the Wazuh VM using the public IP.
```bash
ssh root@your-wazuh-vm-ip
```

2. Update and Upgrade VM
```bash
apt-get update && apt-get upgrade 
```

3. Install Wazuh Manager
- On the Wazuh VM, run the necessary installation commands:
```bash
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
```
- Extract Wazuh Credentials (**Be sure to take note of the credentials. These will be used to log into the Wazuh Dashboard**)
```bash
sudo tar -xvf wazuh-install-files.tar
```

![wazuhinstall](https://github.com/user-attachments/assets/ad7df9bc-0f36-4a98-83c8-8fbf94a65326)

4. Start and Enable Wazuh service
- Start Wazuh:
```bash
systemctl start wazuh-manager
```
- Enable Wazuh to start on boot:
```bash
systemctl enable wazuh-manager
```
- Check that Wazuh is running:
```bash
systemctl status wazuh-manager
```
![image](https://github.com/user-attachments/assets/8635aa1a-80c2-4e92-b6bb-1b1f9c9ff96b)

### TheHive 5 Installation

1. Use SSH to connect to TheHive VM using the public IP.
```bash
ssh root@your-thehive-vm-ip
```

2. Update and Upgrade VM
```bash
apt-get update && apt-get upgrade 
```

3. Install Dependencies
```bash
apt install wget gnupg apt-transport-https git ca-certificates ca-certificates-java curl  software-properties-common python3-pip lsb-release
```

![dependencies](https://github.com/user-attachments/assets/3b295ee8-ef9d-49da-927e-126c4d3d8ab3)

4. Install Java
```bash
wget -qO- https://apt.corretto.aws/corretto.key | sudo gpg --dearmor  -o /usr/share/keyrings/corretto.gpg
echo "deb [signed-by=/usr/share/keyrings/corretto.gpg] https://apt.corretto.aws stable main" |  sudo tee -a /etc/apt/sources.list.d/corretto.sources.list
sudo apt update
sudo apt install java-common java-11-amazon-corretto-jdk
echo JAVA_HOME="/usr/lib/jvm/java-11-amazon-corretto" | sudo tee -a /etc/environment 
export JAVA_HOME="/usr/lib/jvm/java-11-amazon-corretto"
```
![verifyjava](https://github.com/user-attachments/assets/16625625-c401-4a30-8889-2f2db8887b77)

5. Install Cassandra
```bash
wget -qO -  https://downloads.apache.org/cassandra/KEYS | sudo gpg --dearmor  -o /usr/share/keyrings/cassandra-archive.gpg
echo "deb [signed-by=/usr/share/keyrings/cassandra-archive.gpg] https://debian.cassandra.apache.org 40x main" |  sudo tee -a /etc/apt/sources.list.d/cassandra.sources.list
sudo apt update
sudo apt install cassandra
```

6. Install Elasticsearch
```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch |  sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
sudo apt-get install apt-transport-https
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/7.x/apt stable main" |  sudo tee /etc/apt/sources.list.d/elastic-7.x.list
sudo apt update
sudo apt install elasticsearch
```

7. Install TheHive
```bash
wget -O- https://archives.strangebee.com/keys/strangebee.gpg | sudo gpg --dearmor -o /usr/share/keyrings/strangebee-archive-keyring.gpg
echo 'deb [signed-by=/usr/share/keyrings/strangebee-archive-keyring.gpg] https://deb.strangebee.com thehive-5.2 main' | sudo tee -a /etc/apt/sources.list.d/strangebee.list
sudo apt-get update
sudo apt-get install -y thehive
```
### TheHive 5 Configuration

After installing TheHive, perform the following steps to configure it properly and ensure it's ready to receive alerts and integrate with other tools like Wazuh or Cortex.

1. Locate TheHive's main configuration file
```bash
nano /etc/cassandra/cassandra.yaml
```

2. Configure cluster name
```bash
cluster_name: 'vnu'
```
**Replace 'vnu' with your desired cluster name to ensure it matches the Cassandra cluster configuration.**

3. Set listen_address and rpc_address as the public IP of TheHive
- Listen Address:
```bash
listen_address: 209.97.173.238
```

- RPC Address:
```bash
rpc_address: 209.97.173.238
```

4. Configure seed provider
```bash
seeds: "209.97.173.238:7000"
```

5. Stop the Cassandra service
```bash
systemctl stop cassandra.service
```

6. Remove old Cassandra files
```bash
rm -rf /var/lib/cassandra/*
```

7. Start Cassandra service
```bash
systemctl start cassandra.service
```

8. Ensure Cassandra service is running
```bash
systemctl status cassandra.service
```

![image](https://github.com/user-attachments/assets/ea892168-5b12-4ba3-adc7-b2b495124bb1)









