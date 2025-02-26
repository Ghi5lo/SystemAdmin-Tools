# ELK Stack on Windows Server (2019/2022) with Docker

This project will allow us to have a nice and clear view on the **system logs**, providing a setup for deploying the **ELK (Elasticsearch, Logstash, Kibana) stack** on **Windows Server 2019/2022**. 

We're gonna do this by using **Docker and Hyper-V**. 

The guide includes step-by-step instructions to install, configure, and run the ELK stack, as well as integrate **Filebeat** for log collection.

## 📌 Prerequisites

- **Windows Server 2019/2022** with administrator privileges
- **Hyper-V enabled**
- **Docker installed** on Windows Server

## 🚀 Installation Steps

### 1️⃣ Enable Hyper-V
Run the following command in **PowerShell (Admin)**:
```powershell
Install-WindowsFeature -Name Hyper-V -IncludeManagementTools -Restart
```

### 2️⃣ Install Docker
```powershell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
Install-Package -Name docker -ProviderName DockerMsftProvider -Force
Restart-Computer -Force
```

Verify Docker installation:
```powershell
docker version
```

### 3️⃣ Clone the Repository
```powershell
git clone https://github.com/Ghi5lo/SystemAdmin-Tools
cd ELK-LogManagement
```

### 4️⃣ Configure ELK Stack

The repository contains:

- `docker-compose.yml` → Defines the ELK stack setup
- `logstash.conf` → Logstash pipeline configuration

Ensure Docker is running, then start the ELK stack:

```powershell
docker-compose up -d
```
Check running containers:
```powershell
docker ps
```

### 5️⃣ Access Kibana
Once running, open your browser and go to:
```powershell
http://<Server-IP>:5601
```

### 6️⃣ Configure Filebeat (Log Forwarder)
1. Download Filebeat from [Elastic](https://www.elastic.co/downloads/beats/filebeat)
2. Edit `filebeat.yml`:
```yaml
filebeat.inputs:
  - type: log
    paths:
      - C:\Windows\System32\winevt\Logs\Application.evtx
output.logstash:
  hosts: ["<Server-IP>:5044"]
```
3. Start Filebeat:
```powershell
.\filebeat.exe -e
```

## 🛑 Stopping and Cleaning Up
- Stop Elk:
```powershell
docker-compose down
```

- Remove all data (logs, indices):
```powershell
docker-compose down -v
```