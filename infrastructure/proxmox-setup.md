# Proxmox Setup

## Overview
Proxmox VE installed on dedicated second PC to serve as the primary hypervisor for all lab VMs.

---

## Installation

**Date:** April 2026
**Version:** Proxmox VE 8.x (Debian 13 Trixie base)
**Install target:** Kingston HyperX 240GB SATA SSD (sda)
**Filesystem:** ext4

### Installation Steps Taken
1. Downloaded Proxmox VE ISO from proxmox.com
2. Flashed to USB using Balena Etcher
3. Booted from USB on second PC
4. Selected graphical installer
5. Selected ext4 filesystem
6. Configured network settings (see below)
7. Set root password and completed install

### Network Configuration
Configured during install and later corrected via `/etc/network/interfaces`:

```
iface vmbr0 inet static
    address 192.168.1.100/24
    gateway 192.168.1.1
    bridge-ports nic0
    bridge-stp off
    bridge-fd 0
```

**Management interface:** https://192.168.1.100:8006

---

## Post-Install Housekeeping

### 1. Fixed DNS
Added Google DNS to `/etc/resolv.conf` because Proxmox couldn't resolve domain names:
```
nameserver 8.8.8.8
nameserver 8.8.4.4
```

### 2. Fixed System Clock
System clock was showing 2025 instead of 2026, causing SSL certificate failures on apt update.
Manually set time with:
```
date -s "2026-04-03 18:26:00"
```
Installed chrony for ongoing time sync:
```
apt install chrony -y
systemctl enable chrony
systemctl start chrony
```

### 3. Removed Enterprise Repositories
Proxmox ships with paid enterprise repos enabled by default. Removed them and added free community repo:
```
rm /etc/apt/sources.list.d/pve-enterprise.sources
rm /etc/apt/sources.list.d/ceph.sources
echo "deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription" > /etc/apt/sources.list.d/pve-no-subscription.list
```

### 4. System Update
```
apt update
apt dist-upgrade -y
```

---

## Storage Configuration

### sda — Proxmox Boot Drive
Managed automatically by Proxmox installer.

### sdb — Backup Storage
Old Proxmox install from previous machine. Steps to repurpose:
1. Deactivated old LVM volume group: `vgchange -an pve-OLD-BBC9EC96`
2. Wiped drive: `wipefs -a /dev/sdb`
3. Created new GPT partition table via fdisk
4. Formatted as ext4: `mkfs.ext4 /dev/sdb1`
5. Created mount point: `mkdir /mnt/backup`
6. Mounted: `mount /dev/sdb1 /mnt/backup`
7. Added to fstab for auto-mount on reboot:
   `/dev/sdb1 /mnt/backup ext4 defaults 0 2`
8. Added as Directory storage in Proxmox UI (Datacenter > Storage > Add > Directory)

---

## Issues Encountered

| Issue | Cause | Fix |
|-------|-------|-----|
| Network showing DOWN | Ethernet cable not connected | Plugged in cable |
| Wrong IP subnet (192.168.100.x) | Installer defaulted to wrong range | Edited /etc/network/interfaces |
| DNS failure on apt update | No DNS configured | Added 8.8.8.8 to resolv.conf |
| SSL errors on apt update | System clock wrong (showing 2025) | Set time manually, installed chrony |
| Enterprise repo 401 errors | Paid repos enabled by default | Removed .sources files |
| sdb busy during wipe | Old LVM volume group still active | Deactivated with vgchange |
