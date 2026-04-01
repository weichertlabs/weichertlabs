---
title: "Part 1 – Ubuntu Base Setup and Disk Layout"
description: Installing Ubuntu 24.04 LTS bare metal on the WCP machine, partitioning the disks correctly, and preparing the foundation for all services.
date: 2026-03-31
weight: 1
tags: [ubuntu, linux, bare-metal, disk, storage, setup]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

This is the starting point for the WeichertLabs Cloud Platform — installing Ubuntu 24.04 LTS on bare metal and setting up the disk layout that every service in this series will rely on.

---

## Why bare metal Ubuntu?

The RTX 4080 Super is the heart of the WCP setup. It's used by Ollama for AI inference, ComfyUI for image and video generation, and Sunshine for game streaming — all simultaneously.

Running bare metal Ubuntu means all three share the GPU naturally via CUDA, without any passthrough complexity. No Proxmox, no VMs, no driver headaches. The GPU is just there, available to everything.

---

## Requirements

- **Minisforum BD770i** (or any machine with an NVIDIA GPU and 16 GB+ RAM)
- Ubuntu 24.04 LTS ISO — download from [ubuntu.com/download/server](https://ubuntu.com/download/server)
- A USB drive (8 GB+) for the installer
- A tool to write the ISO — [Balena Etcher](https://etcher.balena.io/) or `dd` on macOS/Linux

---

## Step 1 – Create a Bootable USB

**On macOS:**

```bash
# Find the USB drive
diskutil list

# Write the ISO (replace diskN with your USB disk)
sudo dd if=ubuntu-24.04-live-server-amd64.iso of=/dev/rdiskN bs=4m status=progress
```

**Or use Balena Etcher** — select the ISO, select the USB drive, flash.

---

## Step 2 – Boot from USB and Start Installation

1. Insert the USB and power on the machine
2. Enter the boot menu (usually F11, F12, or DEL depending on your machine)
3. Select the USB drive
4. At the Ubuntu boot menu, select **"Try or Install Ubuntu Server"**

---

## Step 3 – Installation Settings

Work through the installer:

**Language and keyboard** — select your preferences

**Network** — Ubuntu will detect your network interface. Configure a static IP if you prefer (recommended for a server):
- IP: e.g. `192.168.1.50`
- Gateway: your router IP
- DNS: `1.1.1.1` or your router

**Storage** — this is the important part. See Step 4 below.

**Profile** — set your username and hostname:
- Name: `patrik` (or your username)
- Server name: `wcp` (or your preferred hostname)
- Username: `patrik`
- Password: choose a strong password

**SSH** — enable **Install OpenSSH server** — you'll want SSH access from day one

**Snaps** — skip all, click Done

---

## Step 4 – Disk Layout

This is the most important step. The WCP setup uses two disks with specific purposes:

```
4TB NVMe  →  OS, all services, AI models, games
2TB HDD   →  Media library (Jellyfin)
```

{{< callout type="warning" >}}
The installer will show all connected disks. Make absolutely sure you select the correct disk for the OS installation. Installing on the wrong disk will erase its contents.
{{< /callout >}}

### NVMe disk — the main disk

In the installer, select **"Custom storage layout"**.

Select the 4TB NVMe disk and create the following partitions:

| Partition | Size | Format | Mount point | Purpose |
|---|---|---|---|---|
| EFI | 1 GB | fat32 | `/boot/efi` | Boot partition |
| Boot | 2 GB | ext4 | `/boot` | Kernel files |
| Root | 100 GB | ext4 | `/` | OS and system files |
| Opt | remaining | ext4 | `/opt` | All Docker services and data |

This keeps the OS lean on the root partition while giving all services their own large partition at `/opt`.

### HDD — media disk

Leave the HDD unformatted during installation. We'll set it up after Ubuntu is installed and mount it at `/mnt/media`.

---

## Step 5 – Complete Installation and Reboot

Click **Done** and confirm the destructive action. The installer will write Ubuntu to the NVMe disk.

When prompted, remove the USB drive and press Enter to reboot.

---

## Step 6 – First Boot and SSH Access

After reboot, Ubuntu will present a login prompt. Log in with the username and password you set during installation.

Get the machine's IP address:

```bash
ip a
```

From your Mac or other machine, SSH in:

```bash
ssh patrik@192.168.1.50
```

All remaining steps in this guide are done over SSH.

---

## Step 7 – System Update

Update everything before doing anything else:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt autoremove -y
```

Reboot if a new kernel was installed:

```bash
sudo reboot
```

---

## Step 8 – Set Up the Media Disk (HDD)

Find the HDD:

```bash
lsblk
```

It will show as `/dev/sda` or `/dev/sdb` — confirm it's the right disk by checking the size.

Get its UUID:

```bash
sudo blkid /dev/sda
```

Format it as ext4:

```bash
sudo mkfs.ext4 /dev/sda
```

Create the mount point:

```bash
sudo mkdir -p /mnt/media
```

Add to `/etc/fstab` for automatic mounting on boot:

```bash
sudo nano /etc/fstab
```

Add this line (replace the UUID with yours):

```
UUID=your-uuid-here  /mnt/media  ext4  defaults  0  2
```

Mount it now without rebooting:

```bash
sudo mount -a
```

Verify:

```bash
df -h | grep media
```

Set permissions:

```bash
sudo chown -R $USER:$USER /mnt/media
```

---

## Step 9 – Create the Directory Structure

Create the folders that will be used throughout this series:

```bash
# AI models and ComfyUI
sudo mkdir -p /mnt/ai/ollama
sudo mkdir -p /mnt/ai/comfyui
sudo chown -R $USER:$USER /mnt/ai

# Games
sudo mkdir -p /mnt/games
sudo chown -R $USER:$USER /mnt/games

# Media (on the HDD)
mkdir -p /mnt/media/films
mkdir -p /mnt/media/series
mkdir -p /mnt/media/music

# Services (Docker will use /opt)
sudo chown -R $USER:$USER /opt
```

---

## Step 10 – Install NVIDIA Drivers

The GPU needs drivers before anything AI or gaming-related can work.

Check if drivers are already detected:

```bash
ubuntu-drivers devices
```

Install the recommended driver:

```bash
sudo ubuntu-drivers autoinstall
```

Or install a specific version:

```bash
sudo apt install -y nvidia-driver-550
```

Reboot:

```bash
sudo reboot
```

Verify after reboot:

```bash
nvidia-smi
```

You should see your RTX 4080 Super listed with the driver version and CUDA version.

---

## What's next

With Ubuntu installed, disks mounted, and NVIDIA drivers running, Part 2 sets up Docker and Tailscale — the two foundations that every service in this series depends on.

**Up next:** Part 2 – Docker and Tailscale *(coming soon)*

---

## Related guides

- [Install Docker and Docker Compose on Linux](/guides/containers/docker/docker-compose-linux/) — install Docker before Part 2
- [Tailscale – Getting Started](/guides/networking/tailscale-getting-started/) — set up remote access
- [Mount a Disk in Linux (fstab)](/guides/operating-systems/linux/mount-disk-linux-fstab/) — more detail on fstab and disk mounting
- [Essential Linux Commands](/guides/operating-systems/linux/essential-linux-commands/) — useful reference while working through this series
