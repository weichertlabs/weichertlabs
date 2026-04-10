---
title: SSH to Windows – Remote Access
description: How to enable OpenSSH Server on Windows, configure SSH key authentication, and connect from macOS or Linux without a password.
date: 2026-04-10
tags: [windows, ssh, remote-access, openssh, powershell, sysadmin]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

Windows 10 and 11 include OpenSSH Server as an optional feature. Once enabled, you can SSH into a Windows machine from any Mac, Linux machine, or other Windows device — using the same tools and workflows you use for Linux servers.

---

## Why SSH to Windows?

- Access a Windows machine remotely without RDP
- Run PowerShell commands from your Mac or Linux terminal
- Transfer files with `scp` or `rsync`
- Automate Windows tasks from scripts on other machines
- No third-party software needed

---

## Step 1 – Install OpenSSH Server

Open PowerShell **as Administrator** and run:

```powershell
# Check if OpenSSH Server is available
Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH.Server*'

# Install OpenSSH Server
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```

Or install via Settings:
1. **Settings → Apps → Optional Features → Add a feature**
2. Search for **OpenSSH Server**
3. Click **Install**

---

## Step 2 – Start the SSH service

```powershell
# Start the service
Start-Service sshd

# Set it to start automatically on boot
Set-Service -Name sshd -StartupType Automatic

# Verify it's running
Get-Service sshd
```

---

## Step 3 – Allow SSH through the firewall

The installer usually creates the firewall rule automatically. Verify it exists:

```powershell
Get-NetFirewallRule -Name *ssh*
```

If missing, create it:

```powershell
New-NetFirewallRule -Name sshd -DisplayName "OpenSSH Server (sshd)" `
    -Enabled True -Direction Inbound -Protocol TCP `
    -Action Allow -LocalPort 22
```

---

## Step 4 – Connect from macOS or Linux

Find the Windows machine's IP address:

```powershell
Get-NetIPAddress -AddressFamily IPv4 |
    Where-Object {$_.InterfaceAlias -notlike "*Loopback*"} |
    Select-Object IPAddress
```

From your Mac or Linux machine:

```bash
ssh username@192.168.1.50
```

Use your Windows username and password when prompted.

{{< callout type="info" >}}
The username is your Windows account name — the one you log in with. For Microsoft accounts, use the local account name or email address format: `user@domain.com`
{{< /callout >}}

---

## Step 5 – Set up SSH key authentication (no password)

SSH keys are more secure and more convenient than passwords. Generate a key pair on your Mac or Linux machine if you haven't already:

```bash
ssh-keygen -t ed25519 -C "your@email.com"
```

### Copy your public key to Windows

**From macOS/Linux:**

```bash
# This won't work directly on Windows — use the manual method below
ssh-copy-id username@192.168.1.50
```

**Manual method (works for standard users):**

```bash
# Copy the public key content
cat ~/.ssh/id_ed25519.pub
```

On the Windows machine, create the authorized_keys file:

```powershell
# Create the .ssh directory
New-Item -ItemType Directory -Path "$env:USERPROFILE\.ssh" -Force

# Create authorized_keys and paste your public key
notepad "$env:USERPROFILE\.ssh\authorized_keys"
```

Paste your public key content and save.

**For Administrator accounts**, the authorized keys file is in a different location:

```powershell
# For accounts in the Administrators group
notepad "C:\ProgramData\ssh\administrators_authorized_keys"
```

Set the correct permissions on the file:

```powershell
# Fix permissions (required for admin accounts)
icacls "C:\ProgramData\ssh\administrators_authorized_keys" /inheritance:r
icacls "C:\ProgramData\ssh\administrators_authorized_keys" /grant "SYSTEM:(F)"
icacls "C:\ProgramData\ssh\administrators_authorized_keys" /grant "BUILTIN\Administrators:(F)"
```

---

## Step 6 – Test key authentication

From your Mac or Linux:

```bash
ssh username@192.168.1.50
```

If the key is set up correctly, it connects without asking for a password.

---

## Step 7 – Configure the default shell

By default, SSH on Windows opens CMD. Change it to PowerShell:

```powershell
# Set PowerShell 7 as the default SSH shell
New-ItemProperty -Path "HKLM:\SOFTWARE\OpenSSH" `
    -Name DefaultShell `
    -Value "C:\Program Files\PowerShell\7\pwsh.exe" `
    -PropertyType String -Force

# Or use built-in Windows PowerShell 5.1
New-ItemProperty -Path "HKLM:\SOFTWARE\OpenSSH" `
    -Name DefaultShell `
    -Value "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" `
    -PropertyType String -Force
```

Restart the SSH service after changing:

```powershell
Restart-Service sshd
```

---

## Copying files with SCP

Once SSH is working, copy files between machines:

```bash
# Copy a file TO Windows
scp localfile.txt username@192.168.1.50:C:\Users\username\Desktop\

# Copy a file FROM Windows
scp username@192.168.1.50:C:\Users\username\Desktop\file.txt ./

# Copy a folder recursively
scp -r /local/folder username@192.168.1.50:C:\Users\username\Documents\
```

---

## SSH config for easy access

On your Mac/Linux, add an entry to `~/.ssh/config`:

```
Host windows-pc
    HostName 192.168.1.50
    User YourWindowsUsername
    IdentityFile ~/.ssh/id_ed25519
```

Now connect with just:

```bash
ssh windows-pc
```

---

## Combine with Tailscale for remote access

With Tailscale installed on both machines, SSH works from anywhere without port forwarding:

```bash
ssh username@100.x.x.x    # via Tailscale IP
ssh windows-pc             # via MagicDNS if configured
```

---

## Troubleshooting

**Connection refused:**
```powershell
# Check if sshd is running
Get-Service sshd

# Check firewall rule
Get-NetFirewallRule -Name *ssh* | Select-Object DisplayName, Enabled, Direction
```

**Permission denied (publickey):**
```powershell
# Check authorized_keys file permissions
icacls "$env:USERPROFILE\.ssh\authorized_keys"
# Should show only the current user with Full Control
```

**Check SSH server logs:**
```powershell
Get-EventLog -LogName Application -Source sshd -Newest 20
```

---

## Related guides

- [SSH Keys – The Right Way](/guides/operating-systems/linux/ssh-keys-linux/) — generating SSH keys on Linux/macOS
- [Tailscale – Getting Started](/guides/networking/tailscale-getting-started/) — secure remote access from anywhere
- [PowerShell – Getting Started](/guides/operating-systems/windows/powershell-getting-started/) — PowerShell basics
- [Windows Terminal – Setup and Tips](/guides/operating-systems/windows/windows-terminal-setup/) — best terminal for Windows
