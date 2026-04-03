# Homelab Documentation

> A self-hosted enterprise simulation environment built for hands-on IT learning.
> Built and maintained by [Your Name] — documenting the journey from zero to functional lab.

---

## What This Is

This repository documents the build, configuration, and ongoing development of my personal homelab. The goal is to simulate enterprise IT infrastructure to develop real-world skills in systems administration, networking, virtualisation, and security.

This is a living document — updated as the lab grows.

---

## Repository Structure

```
homelab-docs/
│
├── README.md                  # This file — overview and index
│
├── hardware/
│   └── inventory.md           # All physical hardware, specs, and roles
│
├── infrastructure/
│   ├── proxmox-setup.md       # Proxmox install, config, and housekeeping
│   ├── network-diagram.md     # Network topology and IP schema
│   └── storage.md             # Storage layout and mount points
│
├── virtual-machines/
│   ├── vm-index.md            # Master list of all VMs and their roles
│   ├── windows-server-dc.md   # Active Directory Domain Controller setup
│   ├── pfsense-firewall.md    # pfSense firewall/router VM
│   └── [add more as built]
│
├── services/
│   ├── active-directory.md    # AD configuration, OUs, GPOs, users
│   ├── dns.md                 # DNS setup and records
│   └── [add more as built]
│
├── troubleshooting/
│   └── issues-log.md          # Problems encountered and how they were solved
│
├── photos/
│   └── [hardware and build photos]
│
└── learning/
    └── notes.md               # Concepts learned, things that clicked, resources
```

---

## Lab Overview

| Component | Details |
|-----------|---------|
| Hypervisor | Proxmox VE |
| Host CPU | AMD Ryzen 5 5600 |
| Host RAM | 8GB DDR4 (upgrading to 16GB) |
| Boot Drive | Kingston HyperX 240GB SATA SSD |
| Backup Drive | Kingston HyperX 240GB SATA SSD |
| Network | Static IP 192.168.1.100, home LAN |

---

## Build Status

- [x] Proxmox installed and configured
- [x] Community repos configured
- [x] Backup storage added
- [ ] First VM — pfSense firewall
- [ ] Windows Server Domain Controller
- [ ] Active Directory configured
- [ ] Client machine joined to domain
- [ ] Monitoring setup

---

## Quick Links

- [Hardware Inventory](hardware/inventory.md)
- [Network Diagram](infrastructure/network-diagram.md)
- [VM Index](virtual-machines/vm-index.md)
- [Troubleshooting Log](troubleshooting/issues-log.md)
