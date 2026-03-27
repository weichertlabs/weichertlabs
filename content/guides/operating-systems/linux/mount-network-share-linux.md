---
title: Mount a Network Share in Linux (fstab)
description: How to permanently mount a Samba (SMB) or NFS network share in Linux using fstab — for Ubuntu, Debian, and Pop!_OS.
date: 2026-03-27
tags: [linux, storage, fstab, samba, nfs, network, ubuntu, debian]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

This guide covers how to mount a Samba (SMB) share or an NFS share permanently in Linux using `/etc/fstab`. Useful for accessing shared folders from a NAS, Windows machine, or another Linux server in your home lab.

---

## Mount a Samba (SMB) Share

### Step 1 – Install the SMB Client

```bash
sudo apt update
sudo apt install -y cifs-utils
```

### Step 2 – Create a Mount Point

```bash
sudo mkdir -p /mnt/nas-share
```

### Step 3 – Create a Credentials File

Store your username and password in a separate file instead of putting them in fstab (which is readable by all users):

```bash
sudo nano /etc/samba/credentials
```

Add:

```
username=yourusername
password=yourpassword
```

Secure the file:

```bash
sudo chmod 600 /etc/samba/credentials
sudo chown root:root /etc/samba/credentials
```

### Step 4 – Add to fstab

```bash
sudo nano /etc/fstab
```

Add this line (replace IP, share name, and your username):

```
//192.168.1.100/sharename  /mnt/nas-share  cifs  credentials=/etc/samba/credentials,uid=1000,gid=1000,iocharset=utf8,_netdev  0  0
```

**What the options mean:**
- `credentials=...` — path to your credentials file
- `uid=1000,gid=1000` — your user and group ID (check with `id` command)
- `iocharset=utf8` — handle special characters correctly
- `_netdev` — wait for network before mounting

### Step 5 – Test

```bash
sudo mount -a
```

Verify:

```bash
df -h | grep nas-share
ls /mnt/nas-share
```

---

## Mount an NFS Share

### Step 1 – Install the NFS Client

```bash
sudo apt update
sudo apt install -y nfs-common
```

### Step 2 – Create a Mount Point

```bash
sudo mkdir -p /mnt/nfs-share
```

### Step 3 – Find Available NFS Exports

Check what the NFS server is sharing:

```bash
showmount -e 192.168.1.100
```

### Step 4 – Test Mount Manually

```bash
sudo mount -t nfs 192.168.1.100:/path/to/share /mnt/nfs-share
```

### Step 5 – Add to fstab

```bash
sudo nano /etc/fstab
```

Add:

```
192.168.1.100:/path/to/share  /mnt/nfs-share  nfs  defaults,_netdev  0  0
```

Test:

```bash
sudo mount -a
```

---

## Troubleshooting

**Mount fails after reboot:**
Make sure `_netdev` is in your fstab options — this tells the system to wait for the network before attempting the mount.

**Permission denied on SMB share:**
Check that `uid` and `gid` match your user. Find your IDs with:

```bash
id
```

**Can't see the NFS export:**
Make sure `nfs-common` is installed and the NFS server allows your IP in its exports config.

---

## Quick Reference

| Command | What it does |
|---|---|
| `sudo mount -a` | Mount all fstab entries |
| `sudo umount /mnt/nas-share` | Unmount a share |
| `df -h` | Show mounted shares and usage |
| `showmount -e 192.168.1.x` | List NFS exports from a server |
| `id` | Show your user and group IDs |

---

## Related Links

- [Mount a Disk in Linux (fstab)](/guides/operating-systems/linux/mount-disk-linux-fstab/) — mount internal and external disks
- [Set Up Samba File Sharing on Linux](/guides/operating-systems/linux/samba-file-sharing-linux/) — create your own Samba share
- [Arch Wiki – fstab](https://wiki.archlinux.org/title/fstab) — detailed fstab documentation
