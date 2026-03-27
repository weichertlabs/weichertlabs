---
title: Install WSL2 and Kali Linux on Windows 11
description: How to install Kali Linux via WSL2 in Windows 11 as part of a local pentest lab — run tools like Nmap, Nikto, ffuf and more without a full VM.
date: 2026-03-27
tags: [windows, wsl2, kali-linux, pentesting, cybersecurity]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

Install Kali Linux via WSL2 in Windows 11 as part of your local pentest lab. This setup lets you run tools like Nmap, Nikto, ffuf, curl, and more — without needing a full virtual machine.

**With WSL2 and Kali you get:**
- Fast setup directly inside Windows
- Seamless access to Linux-based pentesting tools
- Native integration with Docker, Ollama, and other lab components
- Low resource usage — perfect for laptops or VMs

## Video Guide

{{< youtube _g08BgHy2i8 >}}

---

## Step 1 – Enable WSL and Set Version 2

Open **PowerShell as Administrator** and run:

```powershell
wsl --install
```

Make sure WSL2 is set as the default version:

```powershell
wsl --set-default-version 2
```

You may need to reboot after this step.

---

## Step 2 – Install Kali Linux

**Option A – Microsoft Store:**
1. Open the Microsoft Store
2. Search for **"Kali Linux"**
3. Click **Install**

**Option B – Terminal:**
```powershell
wsl --install -d kali-linux
```

---

## Step 3 – Launch Kali for the First Time

```powershell
wsl -d kali-linux
```

You will be prompted to create a Linux username and password.

---

## Step 4 – Verify the Installation

Inside Kali, run:

```bash
uname -a
```

Expected output:

```
Linux kali 5.10.16.3-microsoft-standard-WSL2 ...
```

---

## Step 5 – Update Kali

Before installing any tools, update the package lists and upgrade installed packages:

```bash
sudo apt update && sudo apt upgrade -y
```

Your WSL2 Kali Linux environment is now installed, updated, and ready to use as a local pentesting foundation.

---

## Related Links

- [Kali Linux WSL Documentation](https://www.kali.org/docs/wsl/wsl-preparations/) — official Kali Linux WSL guide
- [Microsoft WSL Documentation](https://learn.microsoft.com/en-us/windows/wsl/) — official Microsoft WSL docs
