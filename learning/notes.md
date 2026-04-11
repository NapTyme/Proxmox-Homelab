# Learning Notes & Documentation Guide

---

## How to Use This Repository

### The Golden Rule
**Document as you go, not after.** Five minutes of notes while something is fresh beats an hour trying to remember later.

---

## What to Write About

### Always document:
- Every command you run that does something significant
- Every config file you edit — include before and after
- Every error message you see, even if you fix it quickly
- Every time something doesn't work the way you expected
- Every time something clicks and you understand a concept

### Write like you're explaining to yourself in 6 months
Don't just write *what* you did — write *why* you did it and *what it means*.

**Bad:** "Ran apt update"
**Good:** "Ran apt update to refresh the package index — this tells the system what versions of software are available from the configured repos, but doesn't actually install anything yet"

---

## When to Take Photos

### Always photograph:
- Physical hardware before and after changes (cable runs, drive installations, RAM changes)
- BIOS/UEFI screens when making configuration changes
- Error screens — especially ones with specific error codes or messages
- Your network diagram drawn on paper before you build it digitally
- The physical setup of your lab space

### Don't bother photographing:
- Normal terminal output that you can copy/paste as text instead
- Successful installs with no interesting output
- Things that are already documented in text

### Photo naming convention:
`YYYY-MM-DD_description.jpg`
Example: `2026-04-03_proxmox-network-config-screen.jpg`

---

## Should I Write About How I Understand Things?

**Yes — absolutely.** This is actually the most valuable part.

When you're explaining a concept in your own words, you're proving you actually understand it rather than just copy-pasting commands. Write entries like:

> "I now understand that vmbr0 is a virtual bridge that sits between the physical NIC (nic0) and the VMs. It's like a virtual switch inside Proxmox — VMs plug into it, and it connects out to the physical network through nic0. This is why even though nic0 shows no IP address, the machine is still reachable at the vmbr0 IP."

These "aha moment" notes are gold for interviews. When someone asks "explain how Proxmox networking works" you'll have already written the answer in your own words.

---

## Learning Notes

### Proxmox Concepts

**Hypervisor:** Software that sits between hardware and virtual machines, allocating CPU/RAM/storage to each VM. Proxmox is a Type 1 (bare metal) hypervisor — it runs directly on hardware, not inside another OS.

**vmbr0 (Virtual Bridge):** Acts like a virtual network switch. VMs connect to it, and it bridges out to the physical network through the real NIC. This is why VMs can get IPs on your home network.

**LVM (Logical Volume Manager):** Linux's way of abstracting physical disks into flexible logical volumes. Proxmox uses LVM by default — pve-root, pve-data, pve-swap are all LVM volumes carved out of the physical drive.

**Static IP vs DHCP:** DHCP means your router assigns an IP automatically (can change). Static means you hardcode the IP in config — essential for a server so it's always at the same address.

**NTP (Network Time Protocol):** Keeps system clocks accurate by syncing to internet time servers. Critical for SSL certificates and logs. Chrony is the NTP client we installed.

---

*Add new concepts here as you learn them*

---

### Flask Production Deployment Stack

**The pattern:** For a Python web app, you don't just run `python app.py` in production. The standard stack is:

```
Browser → Reverse Proxy (Apache/Nginx) → WSGI Server (Gunicorn) → Flask app
```

Each layer has a specific job:

- **Flask** — the application itself. Handles routes, logic, responses. Not designed to serve traffic directly.
- **Gunicorn** — a WSGI server. Translates HTTP requests into something Flask understands, and handles concurrency. Runs on a port (5000 in this case).
- **Apache** — sits on port 80 (standard HTTP) and forwards requests to Gunicorn. The public-facing entry point. Handles SSL termination, security, and routing independently of the app code.

**Netplan** is Ubuntu's network configuration tool. Settings live in YAML files under `/etc/netplan/`. Switching from DHCP to static means setting `dhcp4: no` and manually specifying address, gateway, and DNS. `sudo netplan apply` applies changes without a reboot. You can confirm it worked with `ip a` — a static address shows `valid_lft forever` instead of a DHCP lease timer.

**Systemd services** are how Linux manages persistent background processes — the same way Apache, SSH, and every other system service works. The unit file defines what binary to run, as which user, from which directory, and what to do on crash. Key fields:
- `After=network.target` — don't start until the network is up
- `Restart=always` — auto-recover from any crash
- `WantedBy=multi-user.target` — start at boot in normal multi-user mode

After writing or editing a service file, always run `sudo systemctl daemon-reload` before starting it.

**Python virtual environments** isolate a project's dependencies from the system Python installation. `python3 -m venv venv` creates the environment, `source venv/bin/activate` enters it. Dependencies installed inside only affect that project — prevents version conflicts across multiple apps.
