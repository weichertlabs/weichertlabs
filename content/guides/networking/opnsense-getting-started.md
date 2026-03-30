---
title: OPNsense – Getting Started
description: What OPNsense is, why you would use it, and how to install and configure it as a VM in Proxmox VE — from ISO to working router.
date: 2026-03-30
tags: [networking, opnsense, firewall, router, proxmox, vm]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

OPNsense is an open-source firewall and router platform based on FreeBSD. It is one of the most capable and actively maintained software firewalls available — used in home labs, small businesses, and enterprise environments alike.

**Why OPNsense over a hardware router:**
- Full control over firewall rules, VLANs, routing, and VPN
- Runs as a VM in Proxmox — no dedicated hardware needed
- Web-based UI that is clean and well-documented
- Active development with regular security updates
- Free and open source

---

## Requirements

- Proxmox VE installed (see [Proxmox VE Post Install Script](/guides/virtualization/proxmox/proxmox-ve-post-install/))
- OPNsense ISO downloaded from [opnsense.org/download](https://opnsense.org/download/)
  - Choose: **amd64**, **dvd** image
- At least 2 network interfaces available on your Proxmox host (one for WAN, one for LAN)

---

## Step 1 – Upload the ISO to Proxmox

1. In the Proxmox web interface, go to your storage (e.g. `local`)
2. Click **ISO Images → Upload**
3. Upload the OPNsense ISO

---

## Step 2 – Create the OPNsense VM

1. Click **Create VM** in Proxmox
2. **General** – name it `opnsense`, note the VM ID
3. **OS** – select the OPNsense ISO, set type to **Other**
4. **System** – leave defaults
5. **Disks** – 20 GB is enough for OPNsense itself
   - Bus: **VirtIO Block**
6. **CPU** – 2 cores, type **Host**
7. **Memory** – 2048 MB minimum, 4096 MB recommended
8. **Network** – this is the WAN interface
   - Select your bridge connected to the internet (e.g. `vmbr0`)
   - Model: **VirtIO**
9. **Confirm** – click **Finish**, but do not start yet

### Add the LAN interface

Before starting the VM, add a second network interface for LAN:

1. Select the VM → **Hardware → Add → Network Device**
2. Select your internal bridge (e.g. `vmbr1`)
3. Model: **VirtIO**
4. Click **Add**

---

## Step 3 – Install OPNsense

1. Start the VM and open the **Console**
2. Boot from the ISO — press Enter at the boot menu
3. Log in with: `installer` / `opnsense`
4. Follow the installer:
   - Select **Install (UFS)** for a standard installation
   - Select your disk (vtbd0)
   - Choose **Guided installation**
   - Set a root password when prompted
5. When complete, select **Reboot**
6. Remove the ISO from the VM hardware after reboot

---

## Step 4 – Assign Interfaces

After reboot, OPNsense boots to a console menu.

Select **1 – Assign interfaces:**

- **WAN** → select `vtnet0` (your first network interface — connected to the internet)
- **LAN** → select `vtnet1` (your second interface — internal network)
- Confirm the assignment

---

## Step 5 – Set LAN IP Address

Select **2 – Set interface IP address:**

- Choose **LAN**
- Set a static IP, for example: `192.168.1.1`
- Subnet mask: `24`
- Leave gateway blank (LAN has no upstream gateway)
- Enable DHCP server: **yes**
- DHCP range: `192.168.1.100` to `192.168.1.200`

---

## Step 6 – Access the Web Interface

From a machine connected to the LAN interface, open a browser and go to:

```
https://192.168.1.1
```

Default credentials:
- **Username:** `root`
- **Password:** the password you set during installation

The setup wizard will guide you through basic configuration — timezone, DNS, WAN type (DHCP for most home setups).

---

## Step 7 – Basic Hardening

A few quick settings worth doing after first login:

**Change the web UI port (optional but recommended):**
System → Settings → Administration → TCP port → change from 443 to something less obvious like 8443

**Disable SSH if not needed:**
System → Settings → Administration → uncheck **Enable Secure Shell**

**Enable automatic updates:**
System → Firmware → Settings → enable **Automatic updates**

---

## What's Next

With OPNsense installed and running, you can:
- Add VLANs for network segmentation
- Configure firewall rules between segments
- Set up WireGuard for remote access
- Add Tailscale for easy remote management

For the full VLAN and firewall setup used in the Proxmox Home Lab security series, continue with:
[Part 2 – OPNsense Configuration for the Security Lab](/projects/proxmox-home-lab/manual/part-02-opnsense/)

---

## Related Links

- [OPNsense Documentation](https://docs.opnsense.org) — official docs
- [OPNsense Downloads](https://opnsense.org/download/) — get the latest ISO
- [Tailscale on Proxmox VE](/guides/networking/tailscale-proxmox/) — alternative remote access
- [PCIe Passthrough in Proxmox VE](/guides/virtualization/proxmox/pcie-passthrough-proxmox/) — pass through a physical NIC to OPNsense for better performance
