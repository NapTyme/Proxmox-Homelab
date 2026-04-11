# VM: webserver (ID 100)

**OS:** Ubuntu Server 24.04.4 LTS
**VM ID:** 100
**Role:** Web server — first VM, used for learning and testing
**IP Address:** 192.168.1.120 (DHCP assigned)
**SSH Access:** `ssh naptyme@192.168.1.120`

---

## Creation Settings

### ISO
- Downloaded directly to Proxmox `local` storage via the web UI using the Ubuntu Server ISO URL (Proxmox fetched it directly — no workstation download required)

### Hardware (at install time)
| Setting | Value |
|---------|-------|
| RAM | 3GB (increased from 1GB due to installer crashes — see below) |
| Sockets | 2 |
| CPU Cores | 2 |
| Disk | 16GB |
| Storage | local-lvm |
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
systemctl status qemu-guest-agent.service  # showed inactive — see note below
```

**Important:** The agent service would not start properly until the *QEMU Guest Agent* option was also enabled in the VM's Options tab in the Proxmox UI. The setting appeared in orange after enabling it — orange means the change is pending and will apply after the next restart. After restarting the VM, the status check confirmed the agent was running.

---

## Issues During Setup
See `troubleshooting/issues-log.md` for full entries.

- Ubuntu installer crashed multiple times at 1GB RAM → resolved by temporarily increasing to 3GB / 2 sockets / 2 cores for the install, then scaling back down post-install
