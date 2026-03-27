---
title: Mount a Disk in Linux (fstab)
description: How to find, format, and permanently mount an internal or external disk in Linux using fstab — for Ubuntu, Debian, and Pop!_OS.
date: 2026-03-27
tags: [linux, storage, fstab, mount, disk, ubuntu, debian]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

This guide covers how to find a disk, format it, mount it manually, and make the mount permanent using `/etc/fstab` on Debian-based Linux systems (Ubuntu, Debian, Pop!_OS).

---

## Step 1 – Find Your Disk

List all connected disks and partitions:

```bash
lsblk
```

Example output:

```
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0  500G  0 disk
└─sda1   8:1    0  500G  0 part /
sdb      8:16   0    2T  0 disk
```

Here `sdb` is the new disk with no partitions yet.

---

## Step 2 – Format the Disk

{{< callout type="warning" >}}
Formatting erases all data on the disk. Make absolutely sure you have the right disk before proceeding.
{{< /callout >}}

Create a new partition table and partition:

```bash
sudo fdisk /dev/sdb
```

Inside fdisk:
- Press `g` to create a new GPT partition table
- Press `n` to create a new partition
- Press Enter three times to accept defaults (uses full disk)
- Press `w` to write and exit

Format the partition as ext4:

```bash
sudo mkfs.ext4 /dev/sdb1
```

---

## Step 3 – Create a Mount Point

Create the folder where the disk will be accessible:

```bash
sudo mkdir -p /mnt/storage
```

---

## Step 4 – Mount the Disk Manually

Test the mount before making it permanent:

```bash
sudo mount /dev/sdb1 /mnt/storage
```

Verify it worked:

```bash
df -h | grep storage
```

---

## Step 5 – Make It Permanent with fstab

Find the disk's UUID (never use `/dev/sdb` in fstab — device names can change after reboot):

```bash
sudo blkid /dev/sdb1
```

Example output:

```
/dev/sdb1: UUID="a1b2c3d4-e5f6-7890-abcd-ef1234567890" TYPE="ext4"
```

Copy the UUID, then open fstab:

```bash
sudo nano /etc/fstab
```

Add this line at the bottom (replace the UUID with yours):

```
UUID=a1b2c3d4-e5f6-7890-abcd-ef1234567890  /mnt/storage  ext4  defaults  0  2
```

**What the fields mean:**
- `UUID=...` — the disk identifier
- `/mnt/storage` — where it mounts
- `ext4` — filesystem type
- `defaults` — standard mount options
- `0` — do not dump (backup)
- `2` — fsck order (2 = check after root disk)

---

## Step 6 – Test the fstab Entry

Test without rebooting:

```bash
sudo mount -a
```

If no errors appear, the fstab entry is correct. The disk will now mount automatically on every boot.

---

## Set Permissions (Optional)

If you want your regular user to write to the disk:

```bash
sudo chown -R $USER:$USER /mnt/storage
```

---

## Quick Reference

| Command | What it does |
|---|---|
| `lsblk` | List all disks and partitions |
| `sudo blkid /dev/sdb1` | Show UUID of a partition |
| `sudo fdisk /dev/sdb` | Partition a disk |
| `sudo mkfs.ext4 /dev/sdb1` | Format as ext4 |
| `sudo mount /dev/sdb1 /mnt/storage` | Mount manually |
| `sudo mount -a` | Mount all fstab entries |
| `df -h` | Show mounted disks and usage |

---

## Related Links

- [Mount a Network Share in Linux (fstab)](/guides/operating-systems/linux/mount-network-share-linux/) — mount Samba and NFS shares
- [Set Up Samba File Sharing on Linux](/guides/operating-systems/linux/samba-file-sharing-linux/) — share folders to Windows and macOS
- [Arch Wiki – fstab](https://wiki.archlinux.org/title/fstab) — detailed fstab documentation
