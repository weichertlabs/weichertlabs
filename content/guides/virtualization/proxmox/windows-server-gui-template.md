---
title: Create a Windows Server GUI Template in Proxmox VE
description: How to install VirtIO drivers, run updates, Sysprep, and convert a Windows Server GUI VM into a reusable Proxmox template.
date: 2026-03-27
tags: [proxmox, windows-server, template, sysprep, virtio]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

This is part two of the Windows Server in Proxmox series. This guide walks you through converting a freshly installed Windows Server (GUI) into a reusable Proxmox template — ideal for fast, repeatable VM deployments in lab environments.

**What you'll learn:**
- Installing VirtIO guest tools
- Running full system updates
- Performing Sysprep (generalization)
- Cleaning up ISO mounts
- Converting the VM to a Proxmox template

Once done, you can instantly clone new Windows VMs without repeating the setup process.

## Video Guide

{{< youtube rY2HehLLWhM >}}

---

## Step 1 – Initial Setup

1. After installing Windows Server, set an administrator password
2. Press **Ctrl + Alt + Delete** to log in
3. Adjust the time zone if necessary
4. Open **File Explorer** and navigate to the VirtIO CD/DVD drive
5. Run **virtio-win-guest-tools.exe**
6. Follow the installation steps (usually just Next → Install All) — this installs drivers needed for Proxmox virtual hardware

---

## Step 2 – System Updates

1. Go to **Settings → Update & Security**
2. Let Windows download and install all available updates
3. Click **"Restart now"** once installation is complete
4. After rebooting, log back in and check for remaining updates
5. Repeat until Windows is fully up to date

{{< callout type="info" >}}
If you want specific programs or configurations available on every VM cloned from this template — install and configure them now before proceeding to Sysprep.
{{< /callout >}}

---

## Step 3 – Generalize with Sysprep

Sysprep removes machine-specific information so each cloned VM gets a unique identity.

1. Open **File Explorer** and navigate to:
   ```
   C:\Windows\System32\Sysprep\
   ```
2. Launch **Sysprep.exe**
3. Configure the settings:
   - **System Cleanup Action:** Enter System Out-of-Box Experience (OOBE)
   - **Check:** Generalize
   - **Shutdown Options:** Shutdown
4. Click **OK** and wait for Windows to shut down automatically

---

## Step 4 – Clean Up ISO Mounts (Optional)

Before converting to a template, remove the ISO mounts to avoid boot errors if the ISO files are later deleted.

1. In the Proxmox **Hardware** tab for the VM:
   - Remove one of the CD/DVD drives entirely
   - Edit the remaining one and set it to **"Do not use any media"**

---

## Step 5 – Convert to Template

1. Right-click the Windows Server VM in the Proxmox sidebar
2. Select **"Convert to Template"**
3. Confirm — the VM icon will change to indicate it is now a template

Your Windows Server GUI template is ready. You can now clone it instantly whenever you need a new Windows VM.

---

## Related Links

- [Part 1 – Windows Server Installation in Proxmox VE](/guides/virtualization/proxmox/windows-server-proxmox/) — create the VM and install Windows
- [Proxmox VM Templates Documentation](https://pve.proxmox.com/wiki/VM_Templates_and_Clones) — official Proxmox wiki
- [VirtIO Drivers ISO](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso) — latest stable VirtIO drivers
