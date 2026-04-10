---
title: Essential CMD Commands
description: A practical reference for Windows Command Prompt (CMD) — the commands you still need to know alongside PowerShell, with PowerShell equivalents where relevant.
date: 2026-04-10
tags: [windows, cmd, command-prompt, terminal, reference]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

CMD (Command Prompt) is the original Windows command-line interface. While PowerShell has replaced it for most administrative tasks, CMD is still useful — some tools and scripts require it, and it's always available on any Windows machine, even minimal server installs.

This guide covers the CMD commands you'll actually encounter, with PowerShell equivalents noted where helpful.

---

## Opening CMD

- **Search:** Press `Win+S`, type `cmd`
- **Run dialog:** `Win+R`, type `cmd`, press Enter
- **As Administrator:** Right-click "Command Prompt" → "Run as administrator"
- **From a folder:** Hold `Shift` and right-click in a folder → "Open command window here"

---

## Navigation

| Command | What it does |
|---|---|
| `cd` | Show current directory |
| `cd foldername` | Go into a folder |
| `cd ..` | Go up one level |
| `cd \` | Go to root of current drive |
| `cd /d D:\folder` | Change drive and directory |
| `dir` | List files in current directory |
| `dir /a` | Show hidden files |
| `dir /s` | List recursively |
| `cls` | Clear the screen |

```cmd
REM Go to your user profile
cd %USERPROFILE%

REM Go to a specific path
cd C:\Program Files

REM Switch to D: drive
D:
```

---

## Files and folders

```cmd
REM Create a folder
mkdir MyFolder
md MyFolder

REM Delete a file
del file.txt

REM Delete a folder and all contents
rmdir /s /q MyFolder

REM Copy a file
copy source.txt destination.txt

REM Copy a folder recursively
xcopy C:\Source D:\Destination /E /I /H

REM Move or rename a file
move file.txt C:\NewLocation\
rename oldname.txt newname.txt

REM Show file contents
type file.txt

REM Find text in a file
find "error" logfile.txt

REM Find text case-insensitive
find /I "error" logfile.txt
```

---

## System information

```cmd
REM Computer name and basic info
systeminfo

REM Just the hostname
hostname

REM Windows version
winver
ver

REM Logged in users
query user

REM Running processes
tasklist

REM Kill a process by name
taskkill /F /IM notepad.exe

REM Kill by PID
taskkill /F /PID 1234
```

---

## Networking

```cmd
REM Show IP addresses
ipconfig

REM Full details including DNS
ipconfig /all

REM Flush DNS cache
ipconfig /flushdns

REM Renew DHCP lease
ipconfig /release
ipconfig /renew

REM Test connectivity
ping google.com
ping -n 4 192.168.1.1

REM Trace route
tracert google.com

REM Show active connections
netstat -ano

REM Show listening ports
netstat -an | find "LISTENING"

REM DNS lookup
nslookup google.com

REM Show routing table
route print

REM Map a network drive
net use Z: \\server\share
net use Z: /delete
```

---

## User management

```cmd
REM List local users
net user

REM Create a user
net user labuser Password123! /add

REM Add user to administrators
net localgroup Administrators labuser /add

REM Delete a user
net user labuser /delete

REM List groups
net localgroup

REM List members of a group
net localgroup Administrators
```

---

## Services

```cmd
REM List all services
sc query

REM Check a specific service
sc query wuauserv

REM Start a service
net start wuauserv
sc start wuauserv

REM Stop a service
net stop wuauserv
sc stop wuauserv

REM Set service to auto-start
sc config wuauserv start= auto

REM Set service to disabled
sc config wuauserv start= disabled
```

---

## File comparison and search

```cmd
REM Compare two files
fc file1.txt file2.txt

REM Find files by name
dir /s /b *.log

REM Search in files recursively
findstr /S /I "error" C:\Logs\*.log

REM Find string with line numbers
findstr /N "error" logfile.txt
```

---

## Environment variables

```cmd
REM Show all environment variables
set

REM Show a specific variable
echo %USERNAME%
echo %USERPROFILE%
echo %TEMP%
echo %PATH%

REM Set a variable (current session only)
set MY_VAR=hello

REM Common variables
echo %COMPUTERNAME%    REM Machine name
echo %WINDIR%          REM Windows directory (C:\Windows)
echo %SYSTEMROOT%      REM Same as WINDIR
echo %APPDATA%         REM App data folder
echo %PROGRAMFILES%    REM Program Files
```

---

## Disk and drives

```cmd
REM Show disk usage
chkdsk

REM List drives
fsutil fsinfo drives

REM Check disk for errors (needs restart)
chkdsk C: /F /R

REM Disk cleanup utility
cleanmgr

REM Defragment a drive
defrag C: /U /V
```

---

## Batch file basics

CMD scripts are called batch files (`.bat` or `.cmd`). A simple example:

```cmd
@echo off
REM This is a comment
echo Hello from batch file
echo Current date: %DATE%
echo Current time: %TIME%
pause
```

`@echo off` prevents commands from being printed before they run.
`pause` waits for a key press before closing.

```cmd
REM Variables in batch
set NAME=Patrik
echo Hello, %NAME%

REM If statement
if exist "C:\Logs" (
    echo Logs folder exists
) else (
    mkdir C:\Logs
)

REM For loop
for %%F in (*.txt) do (
    echo Found: %%F
)
```

---

## CMD vs PowerShell equivalents

| CMD | PowerShell | What it does |
|---|---|---|
| `dir` | `Get-ChildItem` / `ls` | List files |
| `cd` | `Set-Location` / `cd` | Change directory |
| `copy` | `Copy-Item` / `cp` | Copy files |
| `del` | `Remove-Item` / `rm` | Delete files |
| `type` | `Get-Content` / `cat` | Show file contents |
| `find` | `Select-String` | Search in files |
| `tasklist` | `Get-Process` | List processes |
| `taskkill` | `Stop-Process` | Kill a process |
| `sc query` | `Get-Service` | List services |
| `net start` | `Start-Service` | Start a service |
| `ipconfig` | `Get-NetIPAddress` | Show IP info |
| `ping` | `Test-Connection` | Test connectivity |
| `netstat` | `Get-NetTCPConnection` | Show connections |

---

## Related guides

- [PowerShell – Getting Started](/guides/operating-systems/windows/powershell-getting-started/) — the modern replacement for CMD
- [PowerShell – Practical Commands for IT](/guides/operating-systems/windows/powershell-practical-commands/) — advanced Windows administration
- [Windows Terminal – Setup and Tips](/guides/operating-systems/windows/windows-terminal-setup/) — run CMD, PowerShell, and WSL side by side
