
# 🛡️ Building My First Mini SOC Lab with Splunk, Sysmon & Windows Forwarder

## 📌 Introduction

In this blog, I share my complete journey building a mini SOC lab to simulate real-world log ingestion and analysis. I used:

- **Splunk Enterprise (Ubuntu)**
- **Splunk Universal Forwarder (Windows 10 & Windows Server)**
- **Sysmon** for rich endpoint visibility
- Manual configurations, CLI testing, and plenty of troubleshooting

This post includes setup steps, mistakes I made, how I fixed them, and how to validate that your pipeline is working.

## 🧰 Lab Setup

### 🔧 Tools Used:

| Tool                       | Purpose                    |
|---------------------------|----------------------------|
| Ubuntu Server 22.04       | Host Splunk Enterprise     |
| Windows Server 2019       | Log forwarding             |
| Windows 10                | Additional endpoint        |
| Sysmon + Swift config     | Advanced telemetry         |
| Splunk Universal Forwarder| Send logs to Splunk        |

## 🏗️ Step-by-Step Setup

### 1️⃣ Install Splunk Enterprise on Ubuntu

```bash
wget -O splunk.tgz 'https://download.splunk.com/products/splunk/releases/10.0.0/linux/splunk-10.0.0-xxxxxxx.tgz'
tar -xvzf splunk.tgz
sudo mv splunk /opt/
sudo /opt/splunk/bin/splunk start --accept-license
```

Enable the forwarder port:

```bash
/opt/splunk/bin/splunk enable listen 9997 -auth admin:yourpassword
```

### 2️⃣ Install Splunk Universal Forwarder on Windows

Download from [Splunk Download Page](https://www.splunk.com/en_us/download/universal-forwarder.html)

Install it normally (GUI installer), then:

Configure `outputs.conf`:

```ini
[tcpout]
defaultGroup = default-autolb-group

[tcpout:default-autolb-group]
server = 192.168.10.131:9997
```

Configure `inputs.conf`:

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

### 3️⃣ Open Firewall Port 9997 (Manual Method)

#### 🧍 Windows: Open via `wf.msc` (GUI)

1. Press `Win + R` → type `wf.msc` → hit Enter  
2. Go to **Inbound Rules** → **New Rule**
3. Select **Port** → **TCP** → enter `9997`
4. Allow the connection → Apply to all profiles
5. Name it `Splunk TCP 9997` → Finish

Repeat the same for **Outbound Rules** if needed.

#### 🐧 Ubuntu (CLI method):

```bash
sudo ufw allow 9997/tcp
sudo ufw reload
```

## 🔍 Testing & Troubleshooting

### ✅ Is Splunk Enterprise Listening?

```bash
netstat -plnt | grep 9997
```

### ✅ Can Forwarder Reach Enterprise?

```powershell
Test-NetConnection 192.168.10.131 -Port 9997
```

### ✅ Is Forwarder Active?

```bash
/opt/splunk/bin/splunk list forward-server
```

### ✅ Are Logs Being Indexed?

```powershell
echo "Test log - $(Get-Date)" >> C:\testlog.txt
```

Search in Splunk:

```spl
index=endpoint source="C:\\testlog.txt"
```

## 🧠 Mistakes I Made

| Mistake | Fix |
|--------|-----|
| Used `.conf.txt` instead of `.conf` | Renamed the file correctly |
| `splunk` command not recognized | Used full path to executable |
| Output not visible in Splunk | Fixed port, firewall, and added test data |
| Forwarder not active | Checked `splunkd.log` and `list forward-server` |
| Missing Sysmon logs | Corrected source path and added XML rendering |

## 📦 My Custom Files

### `outputs.conf`

```ini
[tcpout]
defaultGroup = default-autolb-group

[tcpout:default-autolb-group]
server = 192.168.10.131:9997
```

### `inputs.conf`

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

## ✅ Final Results

- 🪟 Both Windows Server & Windows 10 show in **Data Summary**
- 🐧 Ubuntu Splunk sees forwarders as **Active**
- 📄 `C:\testlog.txt` entries are ingested
- 🔍 Sysmon logs with event codes `1`, `3`, `11` visible
- 🔁 Full log flow from endpoint to SIEM validated!

## 🔮 What’s Next?

- 🧠 Add **Active Directory** logs
- 📊 Build **dashboards & correlation rules**
- 🛠️ Simulate attacks for detection testing

## 🧾 GitHub Project (Coming Soon)

> 💾 I’ll be uploading all config files, screenshots, test logs, and troubleshooting notes [here](https://github.com/your-username/splunk-forwarder-lab)

## 👨‍💻 Follow My SOC Journey

This blog is just the beginning — I’m building a **realistic SOC environment** and documenting every step, every error, and every success.

Stay tuned for:

- AD monitoring
- Custom alerts
- Real-world detection engineering practice

## 📎 Tags

`#Splunk #Sysmon #WindowsForwarder #SOC #SIEM #CyberSecurity #SOCAnalyst #DetectionEngineering #WindowsServer #BlueTeam #LogAnalysis #Homelab #SecurityMonitoring`
