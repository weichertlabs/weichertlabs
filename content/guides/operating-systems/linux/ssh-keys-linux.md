---
title: SSH Keys – The Right Way
description: How to generate SSH keys, copy them to your servers, and disable password authentication for a more secure Linux setup.
date: 2026-03-27
tags: [linux, ssh, security, authentication, ubuntu, debian]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

SSH keys are more secure than passwords and more convenient once set up — no more typing passwords every time you connect to a server. This guide shows you how to generate a key pair, copy it to your servers, and optionally disable password authentication entirely.

## How SSH Keys Work

SSH uses a key pair:
- **Private key** – stays on your local machine, never shared
- **Public key** – copied to the server, safe to share

When you connect, the server checks if your private key matches the public key on file. No password needed.

---

## Step 1 – Generate an SSH Key Pair

Run this on your **local machine** (Mac, Linux, or WSL on Windows):

```bash
ssh-keygen -t ed25519 -C "your@email.com"
```

- `-t ed25519` — modern, fast, and more secure than the older RSA type
- `-C` — a comment to identify the key (your email works well)

You will be asked where to save the key — press **Enter** to accept the default location:

```
~/.ssh/id_ed25519      ← private key (never share this)
~/.ssh/id_ed25519.pub  ← public key (this goes on servers)
```

You will also be asked for a **passphrase** — this encrypts your private key locally. Recommended, but optional.

---

## Step 2 – Copy the Public Key to Your Server

The easiest way:

```bash
ssh-copy-id user@your-server-ip
```

This automatically appends your public key to `~/.ssh/authorized_keys` on the server.

**If ssh-copy-id is not available**, do it manually:

```bash
cat ~/.ssh/id_ed25519.pub | ssh user@your-server-ip "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

---

## Step 3 – Test the Connection

```bash
ssh user@your-server-ip
```

If it connects without asking for a password — the key is working correctly.

---

## Step 4 – Disable Password Authentication (Recommended)

Once your key is working, disable password login to prevent brute-force attacks.

{{< callout type="warning" >}}
Make sure your SSH key login works **before** disabling passwords. If you lock yourself out, you will need console access to recover.
{{< /callout >}}

On the **server**, edit the SSH config:

```bash
sudo nano /etc/ssh/sshd_config
```

Find and change these lines:

```
PasswordAuthentication no
PubkeyAuthentication yes
```

Save and restart SSH:

```bash
sudo systemctl restart sshd
```

Test by opening a **new terminal window** and connecting again — do not close your existing session until you have confirmed it still works.

---

## Step 5 – Managing Multiple Keys (Optional)

If you have multiple servers or accounts, use an SSH config file to manage them:

```bash
nano ~/.ssh/config
```

Example config:

```
Host proxmox
    HostName 192.168.1.10
    User root
    IdentityFile ~/.ssh/id_ed25519

Host ubuntu-server
    HostName 192.168.1.20
    User myuser
    IdentityFile ~/.ssh/id_ed25519
```

Now you can connect with just:

```bash
ssh proxmox
ssh ubuntu-server
```

---

## Quick Reference

| Command | What it does |
|---|---|
| `ssh-keygen -t ed25519` | Generate a new key pair |
| `ssh-copy-id user@host` | Copy public key to server |
| `ssh user@host` | Connect to server |
| `cat ~/.ssh/id_ed25519.pub` | Print your public key |
| `ssh -i ~/.ssh/mykey user@host` | Connect using a specific key |

---

## Related Links

- [OpenSSH Documentation](https://www.openssh.com/manual.html) — official SSH docs
- [Ubuntu SSH Server Guide](https://ubuntu.com/server/docs/service-openssh) — Ubuntu-specific SSH setup
