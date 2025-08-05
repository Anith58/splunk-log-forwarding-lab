
# 🛡️ Splunk Log Forwarding Lab: Windows + Sysmon to Ubuntu

This project sets up a basic SOC-like environment to forward logs from Windows systems (including Sysmon logs) to a Splunk Enterprise server on Ubuntu.

## 🔧 Lab Components

| Component                    | Description                      |
|-----------------------------|----------------------------------|
| Splunk Enterprise            | Installed on Ubuntu Server       |
| Splunk Universal Forwarder  | Installed on Windows 10 & Server |
| Sysmon                      | Installed for advanced telemetry |
| Firewall                    | Manually configured              |

## 🏗️ Setup Overview

### 1️⃣ Splunk Enterprise (Ubuntu)

```bash
wget -O splunk.tgz 'https://download.splunk.com/products/splunk/releases/10.0.0/linux/splunk-10.0.0-xxxxxxx.tgz'

tar -xvzf splunk.tgz

sudo mv splunk /opt/

sudo /opt/splunk/bin/splunk start --accept-license

/opt/splunk/bin/splunk enable listen 9997 -auth admin:yourpassword
```

### 2️⃣ Splunk Universal Forwarder (Windows)

Install from [Splunk Download](https://www.splunk.com/en_us/download/universal-forwarder.html)

#### `outputs.conf`

```ini
[tcpout]
defaultGroup = default-autolb-group

[tcpout:default-autolb-group]
server = 192.168.10.131:9997
```

#### `inputs.conf`

```ini
[WinEventLog://System]
index = endpoint
disabled = false

[WinEventLog://Application]
index = endpoint
disabled = false

[WinEventLog://Security]
index = endpoint
disabled = false

[WinEventLog://Microsoft-Windows-Sysmon/Operational]
index = endpoint
disabled = false
renderXml = false
```

### 3️⃣ Firewall Configuration

#### 🧍 Manual (GUI on Windows)

- Run `wf.msc`
- Add Inbound Rule for TCP 9997
- Allow connection and apply to all profiles

#### 🐧 Ubuntu

```bash
sudo ufw allow 9997/tcp
sudo ufw reload
```

## ✅ Testing

| Check                         | Command/Method                                     |
|------------------------------|----------------------------------------------------|
| Listening on 9997 (Ubuntu)   | `netstat -plnt | grep 9997`                        |
| Port open from Windows       | `Test-NetConnection 192.168.10.131 -Port 9997`     |
| Forwarder active             | `/opt/splunk/bin/splunk list forward-server`       |
| Search log from file         | `echo test >> C:\testlog.txt` then search in UI   |

## 🧠 Lessons Learned

- Misnamed `.conf.txt` files caused issues
- Port/folder permissions prevented forwarding
- Forwarders showed inactive until test data was pushed
- Sysmon required correct source configuration

## 🎯 Future Plans

- Add Active Directory monitoring
- Build detection rules and dashboards
- Simulate attacks for detection engineering

---

## 📎 Tags

`#Splunk #Sysmon #SIEM #SOC #BlueTeam #WindowsServer #DetectionEngineering #Homelab #SecurityMonitoring`

> 💡 Check the `splunk_sysmon_lab.md` file for the full blog-style write-up!
