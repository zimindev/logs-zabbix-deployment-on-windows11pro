# 🚀 **Zabbix 6.4 LTS Deployment on Windows 11 Pro via WSL2 (Ubuntu)**

**🛠️ Monitoring System Deployment Report**  
📅 **Date**: July 7, 2025  
👨‍💻 **Admin**: Sasha Zimin 
🧩 **Zabbix Version**: 6.4 LTS  

---

## 📚 Table of Contents  
1. [🖥️ System Specifications](#️system-specifications)  
2. [⏳ Installation Timeline](#️installation-timeline)  
3. [🔧 Core Installation Process](#️core-installation-process)  
4. [🌐 Web Interface Configuration](#️web-interface-configuration)  
5. [🖥️ Agent Deployment](#️agent-deployment)  
6. [🌍 Website Monitoring Setup](#️website-monitoring-setup)  
7. [✅ Verification & Testing](#️verification--testing)  
8. [📦 Package Manifest](#️package-manifest)  

---

## 🖥️ System Specifications  

| 🔍 Category          | 💡 Specification                  |  
|----------------------|-----------------------------------|  
| **Host OS**          | Windows 11 Pro (Build 23H2)       |  
| **WSL Distro**       | Ubuntu 22.04 LTS                  |  
| **Zabbix Components**| Server 6.4 + Frontend + MariaDB   |  
| **Monitoring Targets**| 3 Windows PCs + 2 Websites       |  
| **Network**          | NAT (WSL2) + Windows Host Bridge  |  

---

## ⏳ Installation Timeline  

| ⏱️ Time      | 📌 Phase                          | ⏳ Duration |  
|-------------|----------------------------------|------------|  
| 10:00-10:15 | WSL2/Ubuntu Setup                | 15 min     |  
| 10:15-10:30 | MariaDB Configuration            | 15 min     |  
| 10:30-10:45 | Zabbix Server Installation       | 15 min     |  
| 10:45-11:00 | Apache/PHP Configuration         | 15 min     |  
| 11:00-11:15 | Web Interface Initial Setup      | 15 min     |  
| 11:15-11:45 | Windows Agents Deployment        | 30 min     |  
| 11:45-12:00 | Website Monitoring Configuration | 15 min     |  

---

## 🔧 Core Installation Process  

### 1️⃣ WSL2 Initialization (10:00)  
```powershell
# Windows Terminal (Admin)
wsl --install -d Ubuntu-22.04
wsl --set-version Ubuntu-22.04 2
```

### 2️⃣ Ubuntu Preparation (10:05)  
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl wget gnupg2
```

### 3️⃣ MariaDB Deployment (10:15)  
```bash
sudo apt install -y mariadb-server mariadb-client
sudo mysql_secure_installation
# Created DB: 'zabbix' | User: 'zabbix'@'localhost'
```

### 4️⃣ Zabbix Repository Setup (10:30)  
```bash
wget https://repo.zabbix.com/zabbix/6.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.4-1+ubuntu22.04_all.deb
sudo dpkg -i zabbix-release_*.deb
sudo apt update
```

### 5️⃣ Core Package Installation (10:35)  
```bash
sudo apt install -y zabbix-server-mysql zabbix-frontend-php \
zabbix-apache-conf zabbix-sql-scripts zabbix-agent
```

### 6️⃣ Database Import (10:40)  
```bash
zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql -uzabbix -p zabbix
```

---

## 🌐 Web Interface Configuration  

### 1️⃣ Apache/PHP Tuning (10:45)  
```bash
sudo nano /etc/php/8.1/apache2/php.ini
# Adjusted:
# memory_limit = 256M
# post_max_size = 32M
# upload_max_filesize = 16M
# max_execution_time = 300
```

### 2️⃣ Service Startup (10:55)  
```bash
sudo systemctl restart zabbix-server zabbix-agent apache2
sudo systemctl enable zabbix-server zabbix-agent apache2
```

### 3️⃣ Firewall Rules (11:00)  
```bash
sudo ufw allow 80/tcp
sudo ufw allow 10050/tcp
```

---

## 🖥️ Agent Deployment  

### 1️⃣ Windows Agents (11:15)  
Downloaded: `zabbix_agent-6.4.7-windows-amd64-openssl.msi`  
**Configuration**:  
```ini
Server=172.28.112.1  # WSL2 Host IP
ServerActive=172.28.112.1
Hostname=PC1-WIN11PRO
```

### 2️⃣ Host Registration in Zabbix  
- Used template: `Template OS Windows by Zabbix agent`  
- Active checks enabled  

---

## 🌍 Website Monitoring Setup  

### 1️⃣ HTTP Test (11:45)  
```yaml
Scenario: "Homepage Availability Check"
URL: https://example.com
Required Status: 200
Timeout: 15s
```

### 2️⃣ SSL Certificate Monitoring  
```yaml
Check: "SSL Expiry"
Host: example.com
Port: 443
Warning: <7d
Critical: <1d
```

---

## ✅ Verification & Testing  

### 1️⃣ Service Status Check  
```bash
systemctl status zabbix-server  # Active (running)
tail -n 20 /var/log/zabbix/zabbix_server.log  # No errors
```

### 2️⃣ Agent Communication Test  
```bash
zabbix_get -s 192.168.1.10 -k system.cpu.util[,idle]
# Returns: 78.3421
```

### 3️⃣ Web Interface Login  
- URL: `http://localhost/zabbix`  
- Credentials: `Admin` / `zabbix`  

---

## 📦 Package Manifest  

### 🔧 Core Components  
```bash
zabbix-server-mysql (6.4.7)
zabbix-frontend-php (6.4.7)
zabbix-agent (6.4.7)
mariadb-server (10.6.12)
apache2 (2.4.52)
php8.1 (8.1.2)
```

### 📡 Networking Tools  
```bash
ufw (0.36.1)
net-tools (1.60+git20181103.0eebece)
```

### 🛠️ Utilities  
```bash
curl (7.81.0)
wget (1.21.2)
nano (6.2)
```

---

## ⚠️ Known Issues & Resolutions  

### ❗ WSL2 IP Changes on Reboot  
**Solution**: Configure static IP via:  
```powershell
# Windows Host:
wsl -d Ubuntu-22.04 -u root ip addr add 172.28.112.10/24 dev eth0
```

### ❗ Zabbix Agent Firewall Block  
**Solution**: Windows Defender rule for `zabbix_agentd.exe`  

---

📄 **Report Generated**: July 7, 2025 – 12:30 EEST  

✅ **Monitoring System Operational**  
- 3/3 Agents reporting  
- 2/2 Websites monitored  
- 15s average data collection interval  

---
💡 `Zabbix + WSL2 = Seamless Windows-Linux Monitoring`
