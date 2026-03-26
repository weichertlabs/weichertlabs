---
title: Proxmox VE Post Install Script
description: Use the community-maintained Proxmox Helper Scripts to apply useful post-install tweaks, clean up repo sources, and update your system.
date: 2026-03-26
tags: [proxmox, post-install, helper-scripts]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

This guide is intended for freshly installed Proxmox VE systems. We'll use the community-maintained **Proxmox Helper Scripts** to apply useful post-install tweaks, clean up the default repository sources, and update the system.

## Video Guide

{{< youtube ogtIbqBz0_U >}}

## What the Script Does

- Removes the enterprise subscription repository notice
- Switches to the no-subscription repository
- Updates package sources
- Applies useful system tweaks
- Optionally updates the entire system

## Step-by-Step Instructions

### Step 1 – Log in as root

You can either use the Proxmox web shell or SSH into your Proxmox node:

```bash
ssh root@your-proxmox-ip
```

### Step 2 – Run the helper script

This script will clean up the default enterprise repo, switch to the no-subscription repo, update package sources, and apply useful tweaks:

```bash
bash <(curl -s https://community-scripts.github.io/ProxmoxVE/scripts/post-pve-install.sh)
```

### Step 3 – Follow the prompts

The script will ask you to confirm changes such as switching repositories and enabling automatic updates. Read each prompt carefully before confirming.

### Step 4 – Update the system

The script will finish by asking if you want to update the system. It is **highly recommended** that you do — this ensures your installation is fully up to date with the latest patches and features.

### Step 5 – Reboot

After completing the update, the script usually prompts for a system reboot. Go ahead and reboot to apply all changes.

```bash
reboot
```

## Related Links

- [Proxmox Helper Scripts](https://community-scripts.github.io/ProxmoxVE/) — community-maintained scripts for Proxmox VE
- [Proxmox VE Documentation](https://pve.proxmox.com/wiki/Main_Page) — official Proxmox documentation
