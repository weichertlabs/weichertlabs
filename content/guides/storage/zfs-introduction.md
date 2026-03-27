---
title: ZFS – Introduction and Key Concepts
description: An introduction to ZFS — what it is, why it's different from traditional filesystems, and why it's a popular choice for home labs and NAS setups.
date: 2026-03-27
tags: [storage, zfs, filesystem, nas, proxmox]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

ZFS is a combined filesystem and volume manager originally developed by Sun Microsystems. It has become one of the most trusted storage solutions for servers, NAS devices, and home labs — and for good reason.

This guide explains what ZFS is, how it works, and why you might want to use it.

---

## What Makes ZFS Different

Traditional filesystems (ext4, NTFS, APFS) manage files. ZFS manages everything — the physical disks, the volumes, and the filesystem — in one unified system.

**Key features:**

**Copy-on-Write (CoW)**
ZFS never overwrites existing data. When you modify a file, ZFS writes the new data to a new location and updates the pointer. Old data stays intact until the new write is confirmed. This means ZFS can never be in an inconsistent state — no need for `fsck` after a crash.

**Data Integrity with Checksums**
Every block of data has a checksum. When ZFS reads data, it verifies the checksum. If corruption is detected, ZFS automatically repairs it using a mirror or RAIDZ copy. Silent data corruption — bit rot — is caught and fixed automatically.

**Snapshots**
A ZFS snapshot is an instant, point-in-time copy of a dataset. Snapshots are nearly free to create (they use no extra space until data changes) and can be used for backups, rollbacks, or cloning.

**Storage Pools (zpools)**
Instead of partitions, ZFS uses storage pools. You add disks to a pool and ZFS manages the space. Adding more disks to a pool expands it automatically.

**Built-in RAID (RAIDZ)**
ZFS has its own RAID implementation — RAIDZ — that avoids the RAID 5 write hole problem that affects hardware RAID controllers.

---

## ZFS Terminology

| Term | What it means |
|---|---|
| **zpool** | A storage pool made up of one or more disks |
| **vdev** | A virtual device — a disk or group of disks in a pool |
| **dataset** | A filesystem or volume inside a pool |
| **snapshot** | A read-only point-in-time copy of a dataset |
| **clone** | A writable copy of a snapshot |
| **scrub** | A background scan that verifies and repairs data integrity |
| **ARC** | Adaptive Replacement Cache — ZFS RAM cache |
| **L2ARC** | Secondary cache on a fast SSD |
| **SLOG/ZIL** | Separate Intent Log — write cache on a fast SSD |

---

## ZFS RAID Levels

| Type | Disks needed | Fault tolerance | Notes |
|---|---|---|---|
| **Single disk** | 1 | None | No redundancy |
| **Mirror** | 2+ | 1 disk failure | Best read performance |
| **RAIDZ1** | 3+ | 1 disk failure | Like RAID 5 but safer |
| **RAIDZ2** | 4+ | 2 disk failures | Like RAID 6 |
| **RAIDZ3** | 5+ | 3 disk failures | Maximum redundancy |

**For home labs:** a mirror (2 disks) is the simplest and most performant option. RAIDZ1 or RAIDZ2 is better for larger setups with 4+ disks.

---

## What ZFS is Not

- **ZFS is not a backup** — RAIDZ protects against disk failure, not against accidental deletion, ransomware, or fire. Always maintain separate backups.
- **ZFS does not like being starved of RAM** — it uses RAM aggressively for caching (ARC). 8 GB+ is recommended, more is better.
- **ZFS pools should not run above 80% capacity** — performance degrades significantly when a pool is nearly full.

---

## Where ZFS is Used

- **TrueNAS** — built entirely around ZFS
- **Proxmox VE** — native ZFS support for VM storage
- **Ubuntu/Debian** — ZFS available as a native filesystem option
- **FreeBSD** — ZFS is the default filesystem
- **Home NAS builds** — ZFS is the go-to choice for DIY NAS setups

---

## Related Links

- [ZFS on Linux (OpenZFS)](/guides/storage/zfs-on-linux/) — install and get started with ZFS on Ubuntu/Debian
- [ZFS Basic Commands](/guides/storage/zfs-basic-commands/) — pools, datasets, snapshots and scrubs
- [ZFS in Proxmox VE](/guides/storage/zfs-in-proxmox/) — how Proxmox integrates ZFS
- [OpenZFS Documentation](https://openzfs.github.io/openzfs-docs/) — official docs
