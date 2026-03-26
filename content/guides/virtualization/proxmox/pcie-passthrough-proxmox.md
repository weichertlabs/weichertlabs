---
title: PCIe Passthrough in Proxmox VE
description: How to configure PCIe passthrough in Proxmox VE — enable IOMMU, bind devices to VFIO, and attach them to a virtual machine.
date: 2026-03-26
tags: [proxmox, pcie, passthrough, gpu, vfio, iommu]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

This guide explains how to configure PCIe passthrough in Proxmox VE using any PCIe device — such as a GPU, USB controller, or network card. As an example, we use an NVIDIA RTX 4080 Super. You will learn how to enable IOMMU, bind the device to VFIO, and attach it to a virtual machine.

## Requirements

- Proxmox VE 7.x or 8.x
- A PCIe device (example: RTX 4080 Super)
- Virtualization (VT-d / AMD-Vi) enabled in BIOS/UEFI

## Video Guide

{{< youtube gf477USzz9o >}}

---

## Step 1 – Enable IOMMU in GRUB

IOMMU (Input-Output Memory Management Unit) allows Proxmox to assign PCIe devices directly to virtual machines. Enable it in the GRUB bootloader:

```bash
nano /etc/default/grub
```

Find the line starting with `GRUB_CMDLINE_LINUX_DEFAULT` and modify it based on your CPU:

**Intel:**
```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt"
```

**AMD:**
```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on iommu=pt"
```

Save with `CTRL+O`, exit with `CTRL+X`, then apply the changes:

```bash
update-grub
```

---

## Step 2 – Load VFIO Kernel Modules

VFIO (Virtual Function I/O) handles PCIe passthrough in a safe and isolated way.

```bash
nano /etc/modules
```

Add these lines at the end of the file:

```
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
```

Save and exit.

---

## Step 3 – Blacklist Host Drivers

Prevent Proxmox from loading drivers for the PCIe device. This is especially important for GPUs:

```bash
nano /etc/modprobe.d/blacklist.conf
```

Add:

```
blacklist nouveau
blacklist nvidia
blacklist nvidiafb
blacklist nvidia_drm
```

This stops Proxmox from using the GPU itself, making it available to the VM.

---

## Step 4 – Bind PCIe Device to VFIO

List your PCI devices to find the ID of your GPU or other device:

```bash
lspci -nn
```

Look for vendor and product IDs, for example:

```
10de:2704, 10de:22bb
```

Create a VFIO config file:

```bash
nano /etc/modprobe.d/vfio.conf
```

Add this line using your device IDs:

```
options vfio-pci ids=10de:2704,10de:22bb
```

Apply the changes:

```bash
update-initramfs -u
```

---

## Step 5 – Reboot

Restart your Proxmox host to apply all changes:

```bash
reboot
```

---

## Step 6 – Verify VFIO Binding

After rebooting, confirm that your device is now using the `vfio-pci` driver:

```bash
lspci -k
```

---

## Step 7 – Add PCIe Device to VM

Open the VM config file (replace `<vmid>` with your VM ID):

```bash
nano /etc/pve/qemu-server/<vmid>.conf
```

Add these lines (replace `01:00.0` and `01:00.1` with your actual device addresses):

```
hostpci0: 01:00.0,pcie=1,x-vga=1
hostpci1: 01:00.1
```

Start the VM — your PCIe device is now passed through and available directly to the virtual machine.

---

## Related Links

- [Proxmox PCI Passthrough Documentation](https://pve.proxmox.com/wiki/PCI_Passthrough) — official Proxmox wiki
- [Proxmox Helper Scripts](https://community-scripts.github.io/ProxmoxVE/) — community-maintained scripts
