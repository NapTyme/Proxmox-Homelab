# VM: webserver (ID 100)

**OS:** Ubuntu Server 24.04.4 LTS
**VM ID:** 100
**Role:** Production deployment of AimOrderChangeRequest — a Flask web app built for workplace truck order amendment processes
**IP Address:** 192.168.1.120 (static — configured via Netplan)
**SSH Access:** `ssh naptyme@192.168.1.120`
**App URL:** http://192.168.1.120
**Source:** https://github.com/NapTyme/AimOrderChangeRequest

---

## Creation Settings

### ISO
- Downloaded directly to Proxmox `local` storage via the web UI using the Ubuntu Server ISO URL (Proxmox fetched it directly — no workstation download required)

### Hardware (at install time)
| Setting | Value |
|---------|-------|
| RAM | 3GB (increased from 1GB due to installer crashes — see troubleshooting log) |
| Sockets | 2 |
| CPU Cores | 2 |
| Disk | 16GB |
| Network Bridge | vmbr0 |

### Hardware (post-install, current)
| Setting | Value |
|---------|-------|
| RAM | 1GB |
| Sockets | 1 |
| CPU Cores | 1 |
| Disk | 16GB |
| Network Bridge | vmbr0 |

---

## Post-Install Steps

### 1. System update
SSHd into the VM from Windows PowerShell:
```
ssh naptyme@192.168.1.120
```
Then ran:
```bash
sudo apt update && sudo apt dist-upgrade
```

### 2. QEMU Guest Agent
Installed and enabled the QEMU guest agent so Proxmox can communicate with the VM:

```bash
sudo apt install qemu-guest-agent
sudo systemctl start qemu-guest-agent.service
systemctl status qemu-guest-agent.service
```

**Important:** The agent service would not start properly until the *QEMU Guest Agent* option was also enabled in the VM's Options tab in the Proxmox UI. The setting appeared in orange after enabling it — orange means the change is pending and will apply after the next restart. After restarting the VM, the status check confirmed the agent was running.

### 3. Apache installed
```bash
sudo apt install apache2
```

---

## Flask App Deployment

### Architecture
```
Browser → Apache (port 80) → Gunicorn (port 5000) → Flask app
```

### 1. Static IP via Netplan
The VM was on DHCP at 192.168.1.120. Converted to static so the address never changes.

Interface confirmed as `ens18` via `ip a`. Config written to `/etc/netplan/50-cloud-init.yaml`:

```yaml
network:
  version: 2
  ethernets:
    ens18:
      dhcp4: no
      addresses:
        - 192.168.1.120/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
```

Applied with:
```bash
sudo netplan apply
```

Confirmed static: `ip a` output changed from `scope global dynamic ens18` with a lease timer to `scope global ens18` with `valid_lft forever`.

### 2. Installed dependencies
```bash
sudo apt update && sudo apt install -y git python3-pip python3-venv
```

### 3. Cloned app from GitHub
```bash
cd /home/naptyme
git clone https://github.com/NapTyme/AimOrderChangeRequest.git
cd AimOrderChangeRequest
```

### 4. Virtual environment and pip install
```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

Dependencies include Flask 3.1.2, Gunicorn 23.0.0, and openpyxl. Environment lives at `/home/naptyme/AimOrderChangeRequest/venv`.

### 5. Tested Gunicorn manually
```bash
gunicorn --bind 0.0.0.0:5000 app:app
```

Confirmed app accessible at `http://192.168.1.120:5000` from the main PC browser, then stopped with Ctrl+C.

### 6. Created systemd service
Runs Gunicorn permanently in the background and survives reboots.

```bash
sudo nano /etc/systemd/system/aimorderchange.service
```

```ini
[Unit]
Description=AimOrderChangeRequest Flask App
After=network.target

[Service]
User=naptyme
WorkingDirectory=/home/naptyme/AimOrderChangeRequest
Environment="PATH=/home/naptyme/AimOrderChangeRequest/venv/bin"
ExecStart=/home/naptyme/AimOrderChangeRequest/venv/bin/gunicorn --bind 0.0.0.0:5000 app:app
Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable aimorderchange
sudo systemctl start aimorderchange
sudo systemctl status aimorderchange
```

See troubleshooting log for an error hit during this step.

### 7. Configured Apache as reverse proxy
Gunicorn runs on port 5000. Apache sits on port 80 and forwards requests through invisibly — users just visit the IP with no port number.

```bash
sudo a2enmod proxy
sudo a2enmod proxy_http
sudo nano /etc/apache2/sites-available/aimorderchange.conf
```

```apache
<VirtualHost *:80>
    ServerName 192.168.1.120

    ProxyPreserveHost On
    ProxyPass / http://127.0.0.1:5000/
    ProxyPassReverse / http://127.0.0.1:5000/
</VirtualHost>
```

```bash
sudo a2ensite aimorderchange.conf
sudo a2dissite 000-default.conf
sudo systemctl reload apache2
```

---

## Outcome

Flask app accessible at `http://192.168.1.120` from any device on the home network. No port number required. Service survives reboots automatically via systemd.

---

## Issues During Setup
See `troubleshooting/issues-log.md` for full entries.

- Ubuntu installer crashed repeatedly at default resources → temporarily increased to 3GB RAM, 2 sockets, 2 cores for install
- QEMU guest agent inactive after install → needed to be enabled in Proxmox Options tab as well as inside the VM
- systemd service failed on first start with `status=203/EXEC` → typo in `ExecStart` path
