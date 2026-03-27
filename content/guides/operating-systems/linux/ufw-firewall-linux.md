---
title: UFW Firewall on Ubuntu and Debian
description: How to set up and manage UFW (Uncomplicated Firewall) on Ubuntu and Debian — allow only the ports you need and block everything else.
date: 2026-03-27
tags: [linux, ufw, firewall, security, ubuntu, debian]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

UFW (Uncomplicated Firewall) is a simple interface for managing firewall rules on Ubuntu and Debian. It wraps the more complex `iptables` tool into easy, readable commands.

The basic principle is simple: **block everything by default, then allow only what you need.**

## Requirements

- Ubuntu 20.04+ or Debian 11+
- SSH access to your server
- sudo privileges

{{< callout type="warning" >}}
Always allow SSH (port 22) **before** enabling UFW. If you enable the firewall without allowing SSH, you will lock yourself out of the server.
{{< /callout >}}

---

## Step 1 – Install UFW

UFW is included by default on Ubuntu. On Debian, install it if needed:

```bash
sudo apt update
sudo apt install -y ufw
```

---

## Step 2 – Set Default Policies

Block all incoming traffic and allow all outgoing traffic by default:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

---

## Step 3 – Allow SSH (Do This First!)

Before enabling the firewall, allow SSH so you don't get locked out:

```bash
sudo ufw allow 22
```

If you run SSH on a custom port (e.g. 2222):

```bash
sudo ufw allow 2222
```

---

## Step 4 – Allow Other Services

Allow only the ports your server actually needs:

```bash
# Web server
sudo ufw allow 80    # HTTP
sudo ufw allow 443   # HTTPS

# Proxmox web interface
sudo ufw allow 8006

# DNS
sudo ufw allow 53

# Custom application port
sudo ufw allow 3000
```

You can also allow by service name instead of port number:

```bash
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https
```

---

## Step 5 – Enable UFW

Once you have allowed SSH, enable the firewall:

```bash
sudo ufw enable
```

You will see a warning about SSH connections — type `y` and press Enter to confirm.

---

## Step 6 – Check the Status

```bash
sudo ufw status verbose
```

Example output:

```
Status: active

To                         Action      From
--                         ------      ----
22                         ALLOW IN    Anywhere
80                         ALLOW IN    Anywhere
443                        ALLOW IN    Anywhere
```

---

## Managing Rules

**Delete a rule:**
```bash
sudo ufw delete allow 80
```

**Allow from a specific IP only:**
```bash
sudo ufw allow from 192.168.1.0/24 to any port 22
```

This allows SSH only from your local network — useful for servers that don't need public SSH access.

**Deny a specific port:**
```bash
sudo ufw deny 3306
```

**Disable UFW (turns off firewall):**
```bash
sudo ufw disable
```

**Reset all rules:**
```bash
sudo ufw reset
```

---

## Quick Reference

| Command | What it does |
|---|---|
| `sudo ufw status verbose` | Show current rules and status |
| `sudo ufw allow 22` | Allow port 22 (SSH) |
| `sudo ufw deny 3306` | Block port 3306 |
| `sudo ufw delete allow 80` | Remove a rule |
| `sudo ufw enable` | Enable the firewall |
| `sudo ufw disable` | Disable the firewall |
| `sudo ufw reset` | Reset all rules |
| `sudo ufw reload` | Reload rules after changes |

---

## Recommended Minimal Setup

For a typical Linux server in a home lab:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22
sudo ufw enable
```

Then add ports as you install services. Only open what you actually need.

---

## Related Links

- [UFW Documentation](https://help.ubuntu.com/community/UFW) — Ubuntu community docs
- [SSH Keys – The Right Way](/guides/operating-systems/linux/ssh-keys-linux/) — secure your SSH access before locking down with UFW
