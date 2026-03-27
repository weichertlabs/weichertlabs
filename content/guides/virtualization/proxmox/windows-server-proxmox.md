---
title: Windows Server Installation in Proxmox VE
description: How to create a Windows Server VM in Proxmox, attach VirtIO drivers, and complete the installation — part one of a two-part series.
date: 2026-03-27
tags: [proxmox, windows-server, vm, virtio, template]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

This guide walks you through how to prepare a clean Windows Server installation in Proxmox VE and convert it into a reusable template — ideal for fast, repeatable VM deployments in lab environments.

This is part one of a two-part series. Once Windows is fully installed, continue with the next guide which covers Sysprep, generalization, and final template preparation.

## Video Guide

{{< youtube ECEdrsYwf8U >}}

## Requirements

- Proxmox VE 7.x or 8.x
- Windows Server ISO downloaded
- VirtIO drivers ISO downloaded from [fedorapeople.org](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso)

---

## Step 1 – Create the Virtual Machine

1. Click **"Create VM"** in the top right corner of the Proxmox interface
2. Under **General** – set a name for your VM and adjust the VM ID if needed
3. In the **OS** tab:
   - Select your Windows Server ISO
   - Set the OS type and version
   - Check **"Add additional drive for VirtIO drivers"** and select your VirtIO ISO
4. In the **System** tab:
   - Choose storage for the VM
   - Check **"Qemu Agent"**
   - Specify TPM storage (optional but recommended)
5. Under **Disks**:
   - Set **Bus/Device** to **VirtIO Block**
   - Set disk size to **50 GiB** (or as preferred)
6. Under **CPU**:
   - Set number of cores (e.g. 2 or 4)
   - Set CPU type to **Host**
7. In the **Memory** tab:
   - Set desired RAM in MiB (e.g. 4096 for 4 GB)
8. Under **Network**:
   - Adjust if needed (e.g. assign to a specific VLAN)
9. Review settings under **Confirm** and click **Finish**

---

## Step 2 – Start the VM and Begin Installation

1. Select your new VM and open the **Console** tab
2. Click **Start Now**
3. When prompted with **"Press any key to boot from CD or DVD…"** — press any key to begin

---

## Step 3 – Install Windows Server

1. Choose your preferred **language, time and keyboard layout**, then click **Next**
2. Click **Install Now**
3. Select your Windows Server edition (e.g. **Standard** or **Datacenter**) and click **Next**
4. Accept the license terms and click **Next**
5. Choose **"Custom: Install Windows only (advanced)"**

---

## Step 4 – Load VirtIO Drivers

Without loading the VirtIO drivers, Windows will not detect the virtual disk. Follow these steps:

1. Click **"Load Driver"** (the icon next to the CD)
2. Click **Browse** and select the **VirtIO ISO**
3. Navigate to the folder:
   ```
   amd64 → [version matching your Windows edition]
   ```
   For example: `2k22` for Windows Server 2022, `w11` for Windows 11
4. Click **OK**
5. Select **Red Hat VirtIO SCSI Controller** and click **Next**
6. After a moment, your storage drive will appear
7. Click **Next** to begin the Windows installation

---

## What's Next

Once Windows Server has finished installing, continue with **Part 2** — Sysprep, generalization, and converting the VM into a reusable Proxmox template.

## Related Links

- [VirtIO Drivers ISO](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso) — latest stable VirtIO drivers
- [Proxmox Windows Guest Agent](https://pve.proxmox.com/wiki/Qemu-guest-agent) — official documentation
- [Proxmox VE Documentation](https://pve.proxmox.com/wiki/Main_Page) — official Proxmox wiki
