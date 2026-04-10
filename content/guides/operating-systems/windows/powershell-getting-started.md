---
title: PowerShell – Getting Started
description: An introduction to PowerShell on Windows — what it is, how it differs from CMD, basic navigation, cmdlets, and setting up your PowerShell profile.
date: 2026-04-10
tags: [windows, powershell, terminal, scripting, sysadmin]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

PowerShell is the modern command-line shell and scripting language for Windows. It replaced CMD as the primary administrative tool and is now cross-platform — it runs on Windows, macOS, and Linux. If you're administering Windows servers or workstations, PowerShell is the tool to learn.

---

## PowerShell vs CMD

| Feature | CMD | PowerShell |
|---|---|---|
| Output type | Plain text | Objects |
| Scripting | Basic batch files | Full scripting language |
| Pipeline | Text only | Object pipeline |
| Remote management | Limited | Built-in (PSRemoting) |
| Cross-platform | No | Yes (PowerShell 7+) |
| Active Directory | No | Full support |

**CMD** still works and some tools require it, but PowerShell can do everything CMD does and much more.

---

## PowerShell versions

- **Windows PowerShell 5.1** — built into Windows 10/11, always available
- **PowerShell 7+** — cross-platform, actively developed, recommended for new work

Install PowerShell 7:

```powershell
winget install Microsoft.PowerShell
```

Or download from [github.com/PowerShell/PowerShell](https://github.com/PowerShell/PowerShell/releases).

Check your version:

```powershell
$PSVersionTable.PSVersion
```

---

## Opening PowerShell

- **Search:** Press `Win+S`, type `PowerShell`
- **Right-click Start menu:** "Windows PowerShell" or "Terminal"
- **Run as Administrator:** Right-click → "Run as administrator" (required for many system commands)
- **Windows Terminal:** Best experience — see the Windows Terminal guide

---

## Basic navigation

PowerShell uses familiar Unix-like aliases for common commands:

| Command | Alias | What it does |
|---|---|---|
| `Get-Location` | `pwd` | Show current directory |
| `Set-Location` | `cd` | Change directory |
| `Get-ChildItem` | `ls`, `dir` | List files and folders |
| `New-Item` | `mkdir` (folders) | Create files or folders |
| `Remove-Item` | `rm`, `del` | Delete files or folders |
| `Copy-Item` | `cp`, `copy` | Copy files |
| `Move-Item` | `mv`, `move` | Move or rename files |
| `Get-Content` | `cat`, `type` | Show file contents |
| `Clear-Host` | `clear`, `cls` | Clear the terminal |

```powershell
# Navigate to your home directory
cd ~

# List all files including hidden
ls -Force

# Create a folder
New-Item -ItemType Directory -Path "C:\Projects\MyProject"

# Create a file
New-Item -ItemType File -Path "readme.txt"
```

---

## Understanding cmdlets

PowerShell commands are called **cmdlets** (pronounced "command-lets"). They follow a `Verb-Noun` naming pattern:

```
Get-Process       # Get information about processes
Stop-Process      # Stop a process
Start-Service     # Start a service
Set-Location      # Set (change) location
New-Item          # Create a new item
Remove-Item       # Remove (delete) an item
```

This consistent naming makes PowerShell predictable — once you know the verbs, you can guess command names.

**Common verbs:**

| Verb | Purpose |
|---|---|
| `Get-` | Retrieve information |
| `Set-` | Change a setting |
| `New-` | Create something |
| `Remove-` | Delete something |
| `Start-` | Start a service/process |
| `Stop-` | Stop a service/process |
| `Invoke-` | Run something |
| `Test-` | Test/check something |

---

## Getting help

PowerShell has built-in documentation:

```powershell
# Get help for a command
Get-Help Get-Process

# Show examples
Get-Help Get-Process -Examples

# Full detailed help
Get-Help Get-Process -Full

# Update help files (run as admin)
Update-Help

# Find commands related to a topic
Get-Command -Noun Service
Get-Command -Verb Get

# Explore what an object looks like
Get-Process | Get-Member
```

---

## The pipeline

PowerShell's pipeline passes **objects** between commands, not just text. This makes filtering and processing much more powerful than CMD.

```powershell
# Get all processes, sort by CPU usage, show top 10
Get-Process | Sort-Object CPU -Descending | Select-Object -First 10

# Get all services that are stopped
Get-Service | Where-Object {$_.Status -eq "Stopped"}

# Get files larger than 100MB
Get-ChildItem C:\ -Recurse | Where-Object {$_.Length -gt 100MB}

# Export to CSV
Get-Process | Export-Csv -Path processes.csv -NoTypeInformation

# Format as a table
Get-Service | Format-Table Name, Status, StartType
```

---

## Variables

```powershell
# Assign a variable
$Name = "Patrik"
$Port = 8080

# Use the variable
Write-Host "Hello, $Name"
Write-Host "Port: $Port"

# Command output to variable
$Processes = Get-Process
$DiskUsage = (Get-PSDrive C).Used

# Arrays
$Servers = @("server01", "server02", "server03")

# Hash tables (like dictionaries)
$Config = @{
    Server = "192.168.1.10"
    Port   = 3389
    User   = "admin"
}
```

---

## Setting up your PowerShell profile

Your PowerShell profile is a script that runs every time you open PowerShell — like `.zshrc` on macOS/Linux.

Find your profile path:

```powershell
$PROFILE
```

Create or edit it:

```powershell
notepad $PROFILE
```

Example profile:

```powershell
# Useful aliases
Set-Alias ll Get-ChildItem
Set-Alias which Get-Command

# Functions
function mkcd {
    param($Path)
    New-Item -ItemType Directory -Path $Path | Out-Null
    Set-Location $Path
}

function uptime {
    (Get-Date) - (gcim Win32_OperatingSystem).LastBootUpTime
}

# Custom prompt showing current directory
function prompt {
    $path = (Get-Location).Path
    "PS $path> "
}

# Welcome message
Write-Host "PowerShell $($PSVersionTable.PSVersion)" -ForegroundColor Cyan
```

Save and reload:

```powershell
. $PROFILE
```

---

## Execution policy

By default, Windows blocks running PowerShell scripts as a security measure.

Check current policy:

```powershell
Get-ExecutionPolicy
```

Allow running local scripts (recommended for personal use):

```powershell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

Policies:
- `Restricted` — no scripts allowed (default)
- `RemoteSigned` — local scripts OK, downloaded scripts need a signature
- `Unrestricted` — all scripts allowed (not recommended)

---

## Related guides

- [PowerShell – Practical Commands for IT](/guides/operating-systems/windows/powershell-practical-commands/) — process, service, and network management
- [SSH to Windows – Remote Access](/guides/operating-systems/windows/ssh-windows-remote-access/) — connect to Windows from any device
- [Windows Terminal – Setup and Tips](/guides/operating-systems/windows/windows-terminal-setup/) — the best terminal experience on Windows
- [Install WSL2 and Kali Linux on Windows 11](/guides/operating-systems/windows/install-wsl2-kali-linux/) — run Linux inside Windows
