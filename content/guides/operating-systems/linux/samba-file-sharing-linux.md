---
title: Set Up Samba File Sharing on Linux
description: How to install and configure Samba on Ubuntu or Debian to share folders with Windows and macOS machines on your local network.
date: 2026-03-27
tags: [linux, samba, file-sharing, network, ubuntu, debian, smb]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

Samba lets your Linux machine share folders with Windows and macOS computers on the same network. This guide covers installation, basic configuration, and connecting from other devices.

---

## Step 1 – Install Samba

```bash
sudo apt update
sudo apt install -y samba
```

Verify it's running:

```bash
sudo systemctl status smbd
```

---

## Step 2 – Create a Shared Folder

Create the folder you want to share:

```bash
sudo mkdir -p /srv/samba/share
sudo chown -R $USER:$USER /srv/samba/share
sudo chmod -R 0775 /srv/samba/share
```

---

## Step 3 – Configure Samba

Back up the original config first:

```bash
sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.bak
```

Open the config file:

```bash
sudo nano /etc/samba/smb.conf
```

Add this block at the bottom of the file:

```ini
[share]
   comment = My Samba Share
   path = /srv/samba/share
   browseable = yes
   read only = no
   valid users = yourusername
   create mask = 0664
   directory mask = 0775
```

**What the options mean:**
- `[share]` — the share name (visible on the network)
- `path` — the folder being shared
- `browseable` — visible when browsing the network
- `read only` — set to `no` to allow writing
- `valid users` — only this user can access the share
- `create mask` — permissions for new files
- `directory mask` — permissions for new folders

---

## Step 4 – Create a Samba User

Samba uses its own password database, separate from Linux system passwords:

```bash
sudo smbpasswd -a yourusername
```

Enter a password when prompted. This is what you'll use when connecting from Windows or macOS.

Enable the user:

```bash
sudo smbpasswd -e yourusername
```

---

## Step 5 – Test the Configuration

Check for syntax errors:

```bash
testparm
```

If no errors, restart Samba:

```bash
sudo systemctl restart smbd
sudo systemctl restart nmbd
```

---

## Step 6 – Allow Samba Through the Firewall

If you have UFW enabled:

```bash
sudo ufw allow samba
```

---

## Step 7 – Connect from Other Devices

**From macOS:**
1. Open Finder
2. Press `CMD+K`
3. Enter: `smb://192.168.1.x/share`
4. Enter your Samba username and password

**From Windows:**
1. Open File Explorer
2. In the address bar type: `\\192.168.1.x\share`
3. Enter your Samba username and password

**From Linux:**
```bash
# Mount the share (see the network share guide for fstab setup)
sudo mount -t cifs //192.168.1.x/share /mnt/share \
  -o username=yourusername,password=yourpassword
```

---

## Multiple Shares Example

You can add as many shares as you need in `smb.conf`:

```ini
[documents]
   path = /srv/samba/documents
   valid users = yourusername
   read only = no

[media]
   path = /srv/samba/media
   valid users = @sambausers
   read only = yes

[public]
   path = /srv/samba/public
   browseable = yes
   read only = no
   guest ok = yes
```

- `@sambausers` — share accessible to a group
- `guest ok = yes` — no password required (use carefully on home networks only)

---

## Useful Commands

| Command | What it does |
|---|---|
| `sudo systemctl restart smbd` | Restart Samba |
| `sudo systemctl status smbd` | Check Samba status |
| `testparm` | Validate smb.conf syntax |
| `sudo smbpasswd -a username` | Add a Samba user |
| `sudo smbpasswd -e username` | Enable a Samba user |
| `sudo smbpasswd -d username` | Disable a Samba user |
| `smbclient -L //localhost -U username` | List local shares |

---

## Related Links

- [Mount a Network Share in Linux (fstab)](/guides/operating-systems/linux/mount-network-share-linux/) — mount this share on other Linux machines
- [UFW Firewall on Ubuntu and Debian](/guides/operating-systems/linux/ufw-firewall-linux/) — manage firewall rules
- [Samba Documentation](https://www.samba.org/samba/docs/) — official Samba docs
