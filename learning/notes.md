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

### Containers vs Virtual Machines

**The core difference — resource handling:**

A VM *allocates* resources to itself — when you give a VM 2 cores and 3GB of RAM, those resources are reserved for that VM. It owns them.

A container sets a *limit* on resources — it can use up to its limit, but those resources aren't exclusively reserved. Multiple containers share the underlying system's resources, and none can exceed their cap.

This difference flows from something deeper: containers share the host's kernel, meaning they don't run a full separate operating system — they're more like isolated processes. VMs, by contrast, run their own complete OS on top of the hypervisor, which is why they need dedicated resources and take longer to boot.

**Migration:**
- VMs can be *live migrated* to another host — moved while running, with no downtime
- Containers must be shut down, moved, and restarted — no live migration

**When to use which:**
- VMs: when you need full OS isolation, Windows, or a different kernel (e.g. running a Windows Server alongside Linux)
- Containers: when you want something lightweight and fast to spin up, and the workload doesn't need a full OS

---

### QEMU Guest Agent

The QEMU guest agent is a small service that runs *inside* the VM and creates a communication channel between the VM's operating system and the Proxmox host. Without it, Proxmox can only see the VM from the outside — with it running, Proxmox can do things like cleanly shut down the guest OS, get the VM's IP address, and freeze the filesystem for consistent snapshots.

**Key lesson from setup:** Even after installing and starting the `qemu-guest-agent` service inside the VM, it won't actually connect unless the feature is also *enabled* in the VM's Options tab in the Proxmox UI. The setting showed in orange after enabling it — orange in Proxmox means the change is pending and will take effect after the next VM restart. After restarting, `systemctl status qemu-guest-agent.service` confirmed it was running.
