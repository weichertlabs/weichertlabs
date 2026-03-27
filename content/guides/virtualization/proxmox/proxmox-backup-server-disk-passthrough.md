---
title: Run Proxmox Backup Server as a VM with Disk Passthrough
description: How to install Proxmox Backup Server as a VM on Proxmox VE 9 and pass through a dedicated physical disk for backup storage.
date: 2026-03-27
tags: [proxmox, pbs, backup, disk-passthrough, vm]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

Running Proxmox Backup Server (PBS) as a VM on the same Proxmox VE host is a practical and cost-effective setup for home labs. Instead of dedicating a separate machine to backups, you pass a physical disk directly through to the PBS VM — giving it exclusive, direct access to the backup storage.

This guide covers the full setup on Proxmox VE 9.x.

## Requirements

- Proxmox VE 9.x
- A dedicated physical disk for backup storage (separate from your OS disk)
- PBS ISO downloaded from [proxmox.com/downloads](https://www.proxmox.com/en/downloads)

{{< callout type="warning" >}}
**Never use your Proxmox OS disk for passthrough.** The disk you pass through to the PBS VM must be a dedicated, separate disk. Passing through the wrong disk will cause data loss.
{{< /callout >}}

---

## Step 1 – Find the Disk ID

Disk passthrough in Proxmox is done using the disk's persistent ID, not `/dev/sdb` — device names can change after a reboot.

SSH into your Proxmox host and list disks by ID:

```bash
ls -l /dev/disk/by-id/
```

Find your backup disk. It will look something like:

```
ata-WDC_WD4000FYYZ_WD-XXXXX -> ../../sdb
```

Note the full ID path — you'll need it in Step 4. Ignore entries ending in `-part1`, `-part2` etc. (those are partitions).

---

## Step 2 – Create the PBS Virtual Machine

1. In the Proxmox web interface, click **"Create VM"**
2. **General** – give it a name (e.g. `pbs`) and a VM ID
3. **OS** – select your PBS ISO, set OS type to **Linux**, kernel **6.x**
4. **System** – leave defaults, optionally enable Qemu Agent
5. **Disks** – create a small boot disk (32 GB is enough for the PBS OS)
   - Set Bus/Device to **VirtIO Block**
6. **CPU** – 2 cores, type **Host**
7. **Memory** – 4096 MB (4 GB) minimum, 8 GB recommended
8. **Network** – connect to your main bridge (e.g. `vmbr0`)
9. **Confirm** – click **Finish**

---

## Step 3 – Add Disk Passthrough via Command Line

Disk passthrough must be configured from the Proxmox host shell — it cannot be done through the web UI.

SSH into your Proxmox host and run (replace the disk ID and VM ID with your own):

```bash
qm set 100 --scsi1 /dev/disk/by-id/ata-WDC_WD4000FYYZ_WD-XXXXX
```

- `100` = your VM ID
- `--scsi1` = the next available SCSI slot (use `scsi1` if `scsi0` is your boot disk)
- The disk ID path from Step 1

Verify it was added correctly:

```bash
qm config 100
```

You should see your passthrough disk listed under `scsi1`.

---

## Step 4 – Install Proxmox Backup Server

1. Start the VM and open the console
2. Boot from the PBS ISO
3. Follow the installer — set hostname (e.g. `pbs.local`), IP address, and password
4. Complete the installation and reboot
5. Remove the ISO from the VM hardware after installation

Access the PBS web interface at:

```
https://your-pbs-ip:8007
```

Log in with `root` and the password you set during installation.

---

## Step 5 – Initialize the Backup Disk in PBS

1. In the PBS web interface, go to **Administration → Disks**
2. Your passthrough disk should appear in the list
3. Select it and click **"Initialize Disk with GPT"**
4. Go to **Administration → Disks → Directory**
5. Click **"Create: Directory"**
   - Select your disk
   - Choose filesystem: **ext4** (recommended for single disk)
   - Set a name for the datastore (e.g. `backup-store`)
6. Click **Create**

---

## Step 6 – Add PBS as a Storage Backend in Proxmox VE

1. In Proxmox VE, go to **Datacenter → Storage → Add → Proxmox Backup Server**
2. Fill in:
   - **ID:** `pbs` (or any name)
   - **Server:** IP address of your PBS VM
   - **Username:** `root@pam`
   - **Password:** your PBS root password
   - **Datastore:** the name you created in Step 5
3. Click **Add**

PBS is now available as a backup target for all your VMs.

---

## Step 7 – Configure a Backup Job

1. In Proxmox VE, go to **Datacenter → Backup → Add**
2. Select:
   - **Storage:** your PBS storage
   - **Schedule:** daily, weekly, or custom
   - **VMs:** select which VMs to back up
   - **Mode:** Snapshot (recommended)
3. Click **Create**

Your VMs will now be backed up automatically to the dedicated disk in your PBS VM.

---

## Tips

- Set a **pruning schedule** in PBS to automatically remove old backups and manage disk space
- PBS supports **deduplication and compression** out of the box — backup storage usage is much lower than raw VM sizes
- You can access backup logs and verify backup integrity directly in the PBS web interface

---

## Related Links

- [Proxmox Backup Server Documentation](https://pbs.proxmox.com/docs/) — official PBS docs
- [Proxmox VE Disk Passthrough Wiki](https://pve.proxmox.com/wiki/Passthrough_Physical_Disk_to_Virtual_Machine_(VM)) — official Proxmox wiki
- [PBS Downloads](https://www.proxmox.com/en/downloads) — latest PBS ISO
