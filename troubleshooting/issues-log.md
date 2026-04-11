# Troubleshooting Log

A running record of every problem encountered in the lab and how it was resolved.
This is one of the most valuable parts of the documentation — future employers want to see problem solving, not just a list of things that worked.

---

## Format

Each entry follows this structure:
- **Date**
- **Problem** — what went wrong, what the symptoms were
- **What I tried** — everything attempted, including things that didn't work
- **Root cause** — what was actually causing it
- **Fix** — what resolved it
- **What I learned** — the takeaway

---

## Log

### 03-04-2026 — Proxmox network interfaces showing DOWN
**Problem:** After fresh Proxmox install, ran `ip a` and both nic0 and vmbr0 showed state DOWN. No network connectivity.
**What I tried:** Checked IP config, re-ran `ip a`
**Root cause:** Ethernet cable was not physically connected to the machine
**Fix:** Plugged ethernet cable into LAN port on router
**What I learned:** Always check physical layer first. `ip a` showing DOWN means no carrier signal detected at the physical level.

---

### 03-04-2026 — Wrong IP subnet after install
**Problem:** Proxmox assigned itself 192.168.100.2 but home network is 192.168.1.x — couldn't reach it from browser
**What I tried:** Checked ip a output
**Root cause:** Installer defaulted to 192.168.100.x range, possibly because network was not connected during install
**Fix:** Edited `/etc/network/interfaces`, changed address to 192.168.1.100/24 and gateway to 192.168.1.1, restarted networking
**What I learned:** Static IP configuration in Linux lives in /etc/network/interfaces. The vmbr0 bridge is what Proxmox uses as its management interface, not the raw nic0.

---

### 03-04-2026 — apt update failing with DNS resolution errors
**Problem:** `apt update` returned "Temporary failure resolving" for all repos
**What I tried:** Checked network connectivity
**Root cause:** No DNS server configured on the Proxmox machine
**Fix:** Added `nameserver 8.8.8.8` and `nameserver 8.8.4.4` to `/etc/resolv.conf`
**What I learned:** DNS and network connectivity are separate things. The machine had a working network route but no way to translate domain names to IPs. /etc/resolv.conf controls DNS resolution on Linux.

---

### 03-04-2026 — SSL certificate verification failures on apt update
**Problem:** After DNS was fixed, apt update still failing with "not live until 2026-03-14" SSL errors
**What I tried:** Re-ran apt update multiple times
**Root cause:** System clock was set to 2025 instead of 2026. SSL certificates have validity windows — if your clock is outside that window, verification fails.
**Fix:** Manually set time with `date -s "2026-04-03 18:26:00"`, then installed chrony for ongoing NTP sync
**What I learned:** System time accuracy is critical for SSL/TLS. This is why servers always run NTP. The RTC (hardware clock) on this machine had drifted significantly — chrony keeps it corrected going forward.

---

### 03-04-2026 — wipefs failing on sdb with "device or resource busy"
**Problem:** Tried to wipe sdb for repurposing but got "probing initialization failed: Device or resource busy"
**What I tried:** Running wipefs directly
**Root cause:** Old Proxmox LVM volume group (pve-OLD-BBC9EC96) from previous install was still active and holding the device
**Fix:** Listed volume groups with `vgs`, deactivated the old one with `vgchange -an pve-OLD-BBC9EC96`, then wipefs succeeded
**What I learned:** LVM volume groups can hold drives busy even if the OS isn't actively using them. Always deactivate VGs before trying to repartition a drive that previously had LVM on it.

---

### 11-04-2026 — Ubuntu Server installer crashing during VM creation
**Problem:** Ubuntu Server installer was crashing repeatedly during installation inside the new VM (ID 100)
**What I tried:** Attempted install with initial low resource allocation
**Root cause:** Insufficient resources allocated to the VM during the installation process — the Ubuntu live installer needs more RAM than the installed system does to run
**Fix:** Temporarily increased VM resources to 3GB RAM, 2 sockets, 2 cores for the duration of the install. After installation completed successfully, scaled back down to 1GB RAM, 1 socket, 1 core
**What I learned:** The resources needed to *run* an OS installer are often higher than what the installed system needs day-to-day. It's good practice to over-provision during install and scale back after.

---

### 11-04-2026 — QEMU guest agent installed but not running
**Problem:** Installed `qemu-guest-agent` inside the VM and attempted to start it, but `systemctl status` showed it as inactive
**What I tried:** `sudo systemctl start qemu-guest-agent.service` — service appeared to start but wasn't functioning
**Root cause:** The QEMU Guest Agent feature was disabled in the VM's Options tab in Proxmox. The guest-side service and the host-side option both need to be enabled for the agent to work
**Fix:** Enabled QEMU Guest Agent in the VM's Options tab in the Proxmox UI, then restarted the VM. The setting showed in orange before restart (Proxmox's indicator for pending changes). After restart, `systemctl status qemu-guest-agent.service` confirmed it was active and running
**What I learned:** Proxmox settings highlighted in orange are pending — they won't take effect until the VM is restarted. Also, software installed inside a VM and features configured on the Proxmox host side are two separate things that both need to be in place.
