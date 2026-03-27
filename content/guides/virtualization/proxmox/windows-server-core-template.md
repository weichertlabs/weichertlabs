---
title: Create a Windows Server Core Template in Proxmox VE
description: How to install VirtIO drivers, configure updates, run Sysprep, and convert a Windows Server Core VM into a reusable Proxmox template.
date: 2026-03-27
tags: [proxmox, windows-server, core, template, sysprep, virtio]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

This guide shows you how to prepare a Windows Server Core VM for templating in Proxmox VE — starting right after the OS installation is complete.

**What you'll learn:**
- Installing VirtIO drivers via PowerShell
- Configuring time zone and system updates
- Running Sysprep for generalization
- Converting the VM into a reusable Proxmox template

This guide is ideal for anyone who regularly deploys Windows Server Core VMs in Proxmox and wants to save time using a pre-configured base image.

## Video Guide

{{< youtube kCEoheKr5SY >}}

---

## Step 1 – Install VirtIO Drivers

Since Windows Server Core has no GUI, everything is done through the command line menu or PowerShell.

1. After installation, set the administrator password
2. Select **option 15** to enter PowerShell
3. Navigate to the VirtIO driver disk (usually the D: drive):
   ```powershell
   D:
   ```
4. List files to confirm you're in the right location:
   ```powershell
   dir
   ```
5. Run the VirtIO installer:
   ```powershell
   .\virtio-win-guest-tools.exe
   ```
6. Follow the installation prompts to install all drivers
7. Once complete, type `exit` to return to the main menu

---

## Step 2 – Change Time Zone

1. From the main menu, select **option 9** (Date and Time)
2. Click **"Change time zone"**
3. Choose your time zone and click **OK** twice

---

## Step 3 – Update the System

1. From the main menu, select **option 5** (Update Settings) and choose **A** for automatic updates
2. Go back to the main menu and select **option 6** (Install Updates)
3. Select **option 1** (All quality updates)
4. When prompted, choose **A** to install all updates
5. When asked to restart, choose **Yes** and wait for the reboot
6. Log in again and go to **option 6** once more to check for remaining updates
7. Repeat until no more updates are available

---

## Step 4 – Generalize with Sysprep

1. From the main menu, select **option 15** (PowerShell)
2. Navigate to the Sysprep folder:
   ```powershell
   cd C:\Windows\System32\Sysprep\
   ```
3. Run Sysprep:
   ```powershell
   .\sysprep.exe
   ```
4. Configure the settings:
   - Check **Generalize**
   - Set **Shutdown Options** to **Shutdown**
5. Click **OK** and wait for Windows Server to shut down automatically

---

## Step 5 – Remove CD Drives and Create Template

1. In Proxmox, select the VM and go to the **Hardware** tab
2. Remove the lower CD/DVD drive — click **Remove**, then **Yes**
3. Edit the remaining CD/DVD drive and set it to **"Do not use any media"**, click **OK**
4. Right-click the VM in the Proxmox sidebar
5. Select **"Convert to Template"** and confirm

Your Windows Server Core template is ready to clone.

---

## Related Links

- [Part 1 – Windows Server Installation in Proxmox VE](/guides/virtualization/proxmox/windows-server-proxmox/) — create the VM and install Windows
- [GUI Template Guide](/guides/virtualization/proxmox/windows-server-gui-template/) — same process for Windows Server with Desktop Experience
- [Proxmox VM Templates Documentation](https://pve.proxmox.com/wiki/VM_Templates_and_Clones) — official Proxmox wiki
- [VirtIO Drivers ISO](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso) — latest stable VirtIO drivers
