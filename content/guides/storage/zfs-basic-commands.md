---
title: ZFS Basic Commands
description: A practical reference for everyday ZFS commands — pools, datasets, snapshots, scrubs, and more.
date: 2026-03-27
tags: [storage, zfs, commands, snapshots, reference]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

A practical reference for the ZFS commands you will use most often. For initial setup see [ZFS on Linux – Installation and Setup](/guides/storage/zfs-on-linux/).

---

## Pool Commands (zpool)

| Command | What it does |
|---|---|
| `zpool list` | List all pools with size and usage |
| `zpool status` | Detailed pool status and health |
| `zpool status mypool` | Status of a specific pool |
| `zpool iostat` | I/O statistics for all pools |
| `zpool iostat mypool 2` | Live I/O stats every 2 seconds |
| `zpool scrub mypool` | Start a data integrity scrub |
| `zpool history mypool` | Show command history for a pool |
| `zpool export mypool` | Safely disconnect a pool |
| `zpool import mypool` | Reconnect an exported pool |
| `zpool destroy mypool` | **Permanently delete a pool and all data** |

---

## Dataset Commands (zfs)

| Command | What it does |
|---|---|
| `zfs list` | List all datasets |
| `zfs list -r mypool` | List all datasets in a pool |
| `zfs create mypool/data` | Create a new dataset |
| `zfs destroy mypool/data` | Delete a dataset |
| `zfs rename mypool/data mypool/storage` | Rename a dataset |
| `zfs get all mypool/data` | Show all properties of a dataset |
| `zfs get used,avail mypool` | Show used and available space |

---

## Properties

```bash
# Enable compression on a dataset
sudo zfs set compression=lz4 mypool/data

# Set a quota (max size a dataset can use)
sudo zfs set quota=500G mypool/data

# Set a reservation (guaranteed space)
sudo zfs set reservation=100G mypool/data

# Change mount point
sudo zfs set mountpoint=/mnt/data mypool/data

# Enable deduplication (use with caution — RAM intensive)
sudo zfs set dedup=on mypool/data

# Check compression ratio
zfs get compressratio mypool/data
```

---

## Snapshots

Snapshots are instant, space-efficient point-in-time copies. They use no extra space until data changes.

```bash
# Create a snapshot
sudo zfs snapshot mypool/data@2026-03-27

# List all snapshots
zfs list -t snapshot

# List snapshots for a specific dataset
zfs list -t snapshot mypool/data

# Roll back to a snapshot (destroys changes made after the snapshot)
sudo zfs rollback mypool/data@2026-03-27

# Delete a snapshot
sudo zfs destroy mypool/data@2026-03-27

# Delete all snapshots older than a specific one
sudo zfs destroy mypool/data@2026-01-01%2026-03-01
```

**Automated snapshots** — install `zfs-auto-snapshot` for automatic scheduled snapshots:

```bash
sudo apt install -y zfs-auto-snapshot
```

This creates hourly, daily, weekly, and monthly snapshots automatically.

---

## Clones

A clone is a writable copy of a snapshot — useful for testing without affecting production data:

```bash
# Create a clone from a snapshot
sudo zfs clone mypool/data@2026-03-27 mypool/data-test

# Delete the clone when done
sudo zfs destroy mypool/data-test
```

---

## Sending and Receiving (Backup)

ZFS can send datasets to another pool or remote server — very efficient for backups:

```bash
# Send a snapshot to another pool (local)
sudo zfs send mypool/data@2026-03-27 | sudo zfs receive backuppool/data

# Send to a remote server via SSH
sudo zfs send mypool/data@2026-03-27 | ssh user@backup-server sudo zfs receive backuppool/data

# Incremental send (only changes since last snapshot — much faster)
sudo zfs send -i mypool/data@2026-03-20 mypool/data@2026-03-27 | \
  sudo zfs receive backuppool/data
```

---

## Scrubs

A scrub reads all data in the pool and verifies checksums — detects and repairs corruption:

```bash
# Start a scrub
sudo zpool scrub mypool

# Check scrub progress
zpool status mypool

# Cancel a running scrub
sudo zpool scrub -s mypool
```

Run scrubs monthly on all your pools. It's the best way to catch disk issues early.

---

## Checking Pool Health

```bash
zpool status
```

Look for the `state` field:

- `ONLINE` — all good
- `DEGRADED` — a disk has failed, pool still works but with reduced redundancy
- `FAULTED` — serious problem, data may be at risk
- `OFFLINE` — a disk is intentionally offline

If a disk fails:

```bash
# See which disk failed
zpool status mypool

# Replace the failed disk
sudo zpool replace mypool /dev/disk/by-id/OLD-DISK-ID /dev/disk/by-id/NEW-DISK-ID

# Watch the resilver (rebuild) progress
zpool status mypool
```

---

## Quick Reference

```bash
# Pool health
zpool status

# Space usage
zpool list
zfs list

# Create snapshot
zfs snapshot mypool/data@$(date +%Y-%m-%d)

# List snapshots
zfs list -t snapshot

# Start scrub
zpool scrub mypool

# Enable compression
zfs set compression=lz4 mypool
```

---

## Related Links

- [ZFS – Introduction and Key Concepts](/guides/storage/zfs-introduction/) — understand ZFS concepts
- [ZFS on Linux – Installation and Setup](/guides/storage/zfs-on-linux/) — get ZFS up and running
- [ZFS in Proxmox VE](/guides/storage/zfs-in-proxmox/) — ZFS in Proxmox
- [OpenZFS Documentation](https://openzfs.github.io/openzfs-docs/) — full command reference
