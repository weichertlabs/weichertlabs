---
title: ZFS in Proxmox VE
description: How Proxmox VE integrates with ZFS — setting up ZFS storage pools, using ZFS for VM disks, and taking advantage of snapshots and replication.
date: 2026-03-27
tags: [proxmox, zfs, storage, vm, snapshots]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

Proxmox VE has native ZFS support built in — no extra packages needed. You can create ZFS pools during installation or add them later, and use them as storage for VM disks, containers, and backups.

If you're new to ZFS, start with the [ZFS Introduction](/guides/storage/zfs-introduction/) first.

---

## ZFS During Proxmox Installation

The easiest way to set up ZFS in Proxmox is during installation. The installer lets you:

1. Select **ZFS** as the filesystem for the OS disk
2. Choose your RAID level (single, mirror, RAIDZ1, RAIDZ2, RAIDZ3)
3. Select which disks to include in the pool

This creates a pool called `rpool` for the OS and sets up ZFS automatically.

---

## Add a ZFS Pool After Installation

If you want to add additional disks as a separate ZFS pool after Proxmox is installed:

### Via the Web Interface

1. Go to **Node → Disks → ZFS**
2. Click **"Create: ZFS"**
3. Give it a name (e.g. `datapool`)
4. Select your RAID level
5. Select the disks to include
6. Click **Create**

The pool is automatically added as a storage backend in Proxmox.

### Via the Command Line

```bash
# Mirror pool with two disks
zpool create datapool mirror \
  /dev/disk/by-id/ata-DISK1-ID \
  /dev/disk/by-id/ata-DISK2-ID

# RAIDZ1 with three disks
zpool create datapool raidz1 \
  /dev/disk/by-id/ata-DISK1-ID \
  /dev/disk/by-id/ata-DISK2-ID \
  /dev/disk/by-id/ata-DISK3-ID
```

Then add it to Proxmox storage:

1. Go to **Datacenter → Storage → Add → ZFS**
2. Select your pool
3. Choose what it can store (disk images, containers, backups)

---

## Recommended ZFS Settings for Proxmox

Enable compression on your pool — it saves significant space for VM disks:

```bash
sudo zfs set compression=lz4 datapool
```

Set the correct ARC size to avoid ZFS consuming too much RAM (leave RAM for VMs):

```bash
# Limit ARC to 8GB (adjust based on your RAM)
echo "options zfs zfs_arc_max=8589934592" | \
  sudo tee /etc/modprobe.d/zfs.conf
sudo update-initramfs -u
```

---

## Using ZFS Storage for VMs

Once your ZFS pool is added as storage:

1. When creating a VM, select your ZFS pool as the storage location for the disk
2. Proxmox creates a ZFS volume (zvol) for each VM disk
3. The VM disk benefits from ZFS checksums, compression, and snapshots automatically

---

## VM Snapshots with ZFS

ZFS snapshots in Proxmox are nearly instant and space-efficient. Unlike full backups, snapshots only track changes.

**Take a snapshot via the web interface:**
1. Select your VM
2. Go to **Snapshots** tab
3. Click **"Take Snapshot"**
4. Give it a name and description
5. Click **OK**

**Via command line:**

```bash
# List VM disks
zfs list | grep vm

# Take a snapshot manually
zfs snapshot datapool/vm-100-disk-0@before-update

# Roll back if something goes wrong
zfs rollback datapool/vm-100-disk-0@before-update
```

---

## ZFS Replication Between Proxmox Nodes

ZFS send/receive is excellent for replicating VM storage between Proxmox nodes:

```bash
# Send a VM disk snapshot to another Proxmox node
zfs send datapool/vm-100-disk-0@snapshot1 | \
  ssh root@192.168.1.11 zfs receive datapool/vm-100-disk-0

# Incremental replication (only changes)
zfs send -i datapool/vm-100-disk-0@snapshot1 \
  datapool/vm-100-disk-0@snapshot2 | \
  ssh root@192.168.1.11 zfs receive datapool/vm-100-disk-0
```

This is a cost-effective way to keep a replica of important VMs on a second Proxmox node.

---

## Monitor ZFS Pool Health in Proxmox

**Via web interface:**
- Go to **Node → Disks → ZFS** — shows pool status and health

**Via command line:**

```bash
# Overall health
zpool status

# Space usage
zpool list
zfs list

# Start a scrub
zpool scrub datapool

# Watch scrub progress
watch -n 5 zpool status datapool
```

Set up a monthly scrub via cron:

```bash
sudo crontab -e
```

Add:

```
0 3 1 * * zpool scrub datapool
```

---

## Useful Commands in Proxmox Context

```bash
# List all ZFS pools
zpool list

# List all VM disks on ZFS
zfs list | grep vm

# Check pool health
zpool status

# List all snapshots
zfs list -t snapshot

# Check compression savings
zfs get compressratio datapool
```

---

## Related Links

- [ZFS – Introduction and Key Concepts](/guides/storage/zfs-introduction/) — understand ZFS before diving in
- [ZFS on Linux – Installation and Setup](/guides/storage/zfs-on-linux/) — ZFS on standard Linux
- [ZFS Basic Commands](/guides/storage/zfs-basic-commands/) — everyday ZFS commands
- [Run Proxmox Backup Server as a VM with Disk Passthrough](/guides/virtualization/proxmox/proxmox-backup-server-disk-passthrough/) — complement ZFS with PBS backups
- [Proxmox ZFS Documentation](https://pve.proxmox.com/wiki/ZFS_on_Linux) — official Proxmox ZFS docs
