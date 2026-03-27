---
title: ZFS on Linux – Installation and Setup
description: How to install OpenZFS on Ubuntu or Debian and create your first storage pool — step by step for beginners.
date: 2026-03-27
tags: [storage, zfs, linux, ubuntu, debian, openzfs]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

This guide covers installing OpenZFS on Ubuntu or Debian and creating your first storage pool. If you're new to ZFS, start with the [ZFS Introduction](/guides/storage/zfs-introduction/) first.

## Requirements

- Ubuntu 20.04+ or Debian 11+
- At least one dedicated disk for ZFS (separate from your OS disk)
- At least 8 GB RAM (more is better for ZFS caching)

---

## Step 1 – Install OpenZFS

```bash
sudo apt update
sudo apt install -y zfsutils-linux
```

Verify the installation:

```bash
zfs --version
```

---

## Step 2 – Find Your Disks

List available disks:

```bash
lsblk
```

Get persistent disk IDs (always use IDs in ZFS, not `/dev/sdb`):

```bash
ls -l /dev/disk/by-id/
```

Note the IDs of the disks you want to use. They look like:
```
ata-WDC_WD4000FYYZ_WD-XXXXX
```

---

## Step 3 – Create a Storage Pool

**Single disk (no redundancy — lab use only):**

```bash
sudo zpool create mypool /dev/disk/by-id/ata-YOUR-DISK-ID
```

**Mirror (2 disks — recommended minimum for data you care about):**

```bash
sudo zpool create mypool mirror \
  /dev/disk/by-id/ata-DISK1-ID \
  /dev/disk/by-id/ata-DISK2-ID
```

**RAIDZ1 (3+ disks — 1 disk fault tolerance):**

```bash
sudo zpool create mypool raidz1 \
  /dev/disk/by-id/ata-DISK1-ID \
  /dev/disk/by-id/ata-DISK2-ID \
  /dev/disk/by-id/ata-DISK3-ID
```

**RAIDZ2 (4+ disks — 2 disk fault tolerance):**

```bash
sudo zpool create mypool raidz2 \
  /dev/disk/by-id/ata-DISK1-ID \
  /dev/disk/by-id/ata-DISK2-ID \
  /dev/disk/by-id/ata-DISK3-ID \
  /dev/disk/by-id/ata-DISK4-ID
```

---

## Step 4 – Verify the Pool

```bash
zpool status
```

Example output:

```
  pool: mypool
 state: ONLINE
config:
        NAME        STATE     READ WRITE CKSUM
        mypool      ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            sdb     ONLINE       0     0     0
            sdc     ONLINE       0     0     0
```

Check available space:

```bash
zpool list
```

---

## Step 5 – Create Datasets

Datasets are like folders inside a pool, but with their own properties (compression, quotas, snapshots):

```bash
# Create a dataset
sudo zfs create mypool/data
sudo zfs create mypool/backups
sudo zfs create mypool/media
```

List datasets:

```bash
zfs list
```

Datasets are automatically mounted at `/mypool/data`, `/mypool/backups` etc.

---

## Step 6 – Enable Compression

ZFS compression is highly recommended — it's fast and saves significant disk space:

```bash
sudo zfs set compression=lz4 mypool
```

LZ4 is the recommended compression algorithm — fast with good compression ratio. Enable it on the pool level and all datasets inherit it automatically.

Check compression ratio after some data is written:

```bash
zfs get compressratio mypool
```

---

## Step 7 – Set Up Regular Scrubs

A scrub verifies and repairs data integrity. Run it monthly:

```bash
sudo zpool scrub mypool
```

Check scrub status:

```bash
zpool status mypool
```

To automate monthly scrubs, add a cron job:

```bash
sudo crontab -e
```

Add:

```
0 2 1 * * zpool scrub mypool
```

This runs a scrub at 2am on the 1st of every month.

---

## Set Mount Point and Permissions

Change where a dataset is mounted:

```bash
sudo zfs set mountpoint=/mnt/data mypool/data
```

Set permissions:

```bash
sudo chown -R $USER:$USER /mnt/data
```

---

## Related Links

- [ZFS – Introduction and Key Concepts](/guides/storage/zfs-introduction/) — understand ZFS before diving in
- [ZFS Basic Commands](/guides/storage/zfs-basic-commands/) — snapshots, clones, and everyday commands
- [ZFS in Proxmox VE](/guides/storage/zfs-in-proxmox/) — ZFS integration in Proxmox
- [OpenZFS Documentation](https://openzfs.github.io/openzfs-docs/) — official docs
