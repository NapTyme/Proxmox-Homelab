# Hardware Inventory

Last updated: April 2026

---

## Proxmox Server (Primary Hypervisor)

| Component | Details |
|-----------|---------|
| CPU | AMD Ryzen 5 5600 |
| RAM | 8GB DDR4 (single stick, right slot only — left slot dead) |
| Boot Drive | Kingston HyperX Fury 240GB SATA SSD (sda) |
| Backup Drive | Kingston HyperX Fury 240GB SATA SSD (sdb) |
| Storage Drive | 931GB NVMe (currently unused — old Windows install) |
| Motherboard | ASRock A320M-HDV (2x DIMM slots, 4x SATA, 1x M.2) |
| GPU | NVIDIA RTX 3060 (unused by Proxmox) |
| Network | Realtek Gigabit LAN — static IP 192.168.1.100 |
| OS | Proxmox VE (Debian 13 Trixie base) |

### Notes
- Left DIMM slot is non-functional — only right slot works
- RAM upgrade to 16GB planned
- NVMe drive to be repurposed as VM storage once Windows no longer needed

---

## Main Workstation

| Component | Details |
|-----------|---------|
| CPU | AMD Ryzen 5 5600X |
| RAM | 32GB DDR4 3600MHz |
| Boot Drive | 500GB NVMe SSD |
| Storage Drive | 2TB NVMe SSD |
| GPU | NVIDIA RTX 3060 OC 12GB |
| OS | Windows 11 |
| Role | Daily driver, Proxmox management via browser |

---

## Endpoint Devices (Test Machines)

| Device | Specs | Role |
|--------|-------|------|
| Craptop 1 | TBD | Physical endpoint / malware testing |
| Craptop 2 | TBD | Physical endpoint / domain client testing |

---

## Network Equipment

| Device | Details |
|--------|---------|
| Router/Modem | DSL router — gateway 192.168.1.1 |
| DNS | 8.8.8.8 / 8.8.4.4 (Google) configured on Proxmox |
