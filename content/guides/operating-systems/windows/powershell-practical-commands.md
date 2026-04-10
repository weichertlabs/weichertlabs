---
title: PowerShell – Practical Commands for IT
description: Essential PowerShell commands for IT administration — processes, services, networking, file management, remote management, and system information on Windows.
date: 2026-04-10
tags: [windows, powershell, sysadmin, it, networking, services]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

A practical reference for IT administration with PowerShell. These are the commands you reach for when managing Windows systems — servers, workstations, and remote machines.

---

## System information

```powershell
# OS version and build
Get-ComputerInfo | Select-Object OsName, OsVersion, OsBuildNumber

# Hardware summary
Get-ComputerInfo | Select-Object CsName, CsNumberOfProcessors, CsTotalPhysicalMemory

# Uptime
(Get-Date) - (Get-CimInstance Win32_OperatingSystem).LastBootUpTime

# BIOS and hardware
Get-CimInstance Win32_BIOS
Get-CimInstance Win32_Processor | Select-Object Name, NumberOfCores, MaxClockSpeed

# Installed RAM
Get-CimInstance Win32_PhysicalMemory | Select-Object Capacity, Speed

# Disk information
Get-PSDrive -PSProvider FileSystem
Get-Disk
Get-Volume
```

---

## Process management

```powershell
# List all processes
Get-Process

# Filter by name
Get-Process -Name chrome

# Sort by memory usage
Get-Process | Sort-Object WorkingSet -Descending | Select-Object -First 10 Name, WorkingSet, CPU

# Kill a process by name
Stop-Process -Name notepad

# Kill by PID
Stop-Process -Id 1234

# Kill all instances
Get-Process chrome | Stop-Process

# Start a process
Start-Process notepad.exe

# Start as administrator
Start-Process powershell -Verb RunAs
```

---

## Service management

```powershell
# List all services
Get-Service

# Filter by status
Get-Service | Where-Object {$_.Status -eq "Running"}
Get-Service | Where-Object {$_.Status -eq "Stopped"}

# Check a specific service
Get-Service -Name wuauserv

# Start / stop / restart
Start-Service -Name wuauserv
Stop-Service -Name wuauserv
Restart-Service -Name spooler

# Enable/disable autostart
Set-Service -Name wuauserv -StartupType Automatic
Set-Service -Name wuauserv -StartupType Disabled

# Wait for a service to start
(Get-Service -Name wuauserv).WaitForStatus("Running", "00:00:30")
```

---

## Networking

```powershell
# Show all IP addresses
Get-NetIPAddress | Select-Object InterfaceAlias, IPAddress, AddressFamily

# Show network adapters
Get-NetAdapter | Select-Object Name, Status, LinkSpeed, MacAddress

# Show routing table
Get-NetRoute

# Test connectivity (like ping)
Test-Connection google.com
Test-Connection google.com -Count 4 -Quiet   # returns True/False

# Test if a port is open
Test-NetConnection -ComputerName 192.168.1.10 -Port 3389
Test-NetConnection -ComputerName google.com -Port 443

# Show active connections
Get-NetTCPConnection | Where-Object {$_.State -eq "Established"}

# Show listening ports
Get-NetTCPConnection -State Listen | Select-Object LocalAddress, LocalPort, OwningProcess

# DNS lookup
Resolve-DnsName google.com
Resolve-DnsName weichertlabs.com -Type MX

# Flush DNS cache
Clear-DnsClientCache

# Show DNS cache
Get-DnsClientCache

# Get network statistics
netstat -ano   # still useful from within PowerShell
```

---

## File and folder management

```powershell
# List files with details
Get-ChildItem -Path C:\Logs -File | Select-Object Name, Length, LastWriteTime

# Recursive search for files
Get-ChildItem -Path C:\ -Recurse -Filter "*.log" -ErrorAction SilentlyContinue

# Find large files
Get-ChildItem -Path C:\ -Recurse -ErrorAction SilentlyContinue |
    Where-Object {$_.Length -gt 100MB} |
    Sort-Object Length -Descending |
    Select-Object FullName, @{Name="SizeMB";Expression={[math]::Round($_.Length/1MB,2)}}

# Copy a folder recursively
Copy-Item -Path C:\Source -Destination D:\Backup -Recurse

# Delete files older than 30 days
Get-ChildItem -Path C:\Logs -Recurse |
    Where-Object {$_.LastWriteTime -lt (Get-Date).AddDays(-30)} |
    Remove-Item -Force

# Get file hash (for integrity checking)
Get-FileHash C:\Setup.exe -Algorithm SHA256

# Search file contents
Select-String -Path C:\Logs\*.log -Pattern "error" -CaseSensitive:$false
```

---

## Windows Event Log

```powershell
# Show recent system errors
Get-EventLog -LogName System -EntryType Error -Newest 20

# Show application warnings and errors
Get-EventLog -LogName Application -EntryType Error,Warning -Newest 50

# Filter by event ID
Get-EventLog -LogName Security -InstanceId 4625 -Newest 10  # Failed logins

# Using newer Get-WinEvent (more powerful)
Get-WinEvent -LogName System -MaxEvents 50 |
    Where-Object {$_.LevelDisplayName -eq "Error"}

# Search all logs for a keyword
Get-WinEvent -LogName Application |
    Where-Object {$_.Message -like "*connection refused*"} |
    Select-Object TimeCreated, Message
```

---

## User and group management

```powershell
# List local users
Get-LocalUser

# Create a local user
New-LocalUser -Name "labuser" -Password (ConvertTo-SecureString "P@ssw0rd!" -AsPlainText -Force)

# Add user to a group
Add-LocalGroupMember -Group "Administrators" -Member "labuser"

# List local groups
Get-LocalGroup

# List members of a group
Get-LocalGroupMember -Group "Administrators"

# Disable a user account
Disable-LocalUser -Name "labuser"

# Remove a user
Remove-LocalUser -Name "labuser"
```

---

## Windows Firewall

```powershell
# Show firewall status
Get-NetFirewallProfile | Select-Object Name, Enabled

# Enable/disable firewall for all profiles
Set-NetFirewallProfile -All -Enabled True
Set-NetFirewallProfile -All -Enabled False

# List firewall rules
Get-NetFirewallRule | Where-Object {$_.Enabled -eq "True"} |
    Select-Object DisplayName, Direction, Action

# Add a firewall rule (allow RDP)
New-NetFirewallRule -DisplayName "Allow RDP" `
    -Direction Inbound -Protocol TCP `
    -LocalPort 3389 -Action Allow

# Add a rule to allow a specific port
New-NetFirewallRule -DisplayName "Allow HTTP" `
    -Direction Inbound -Protocol TCP `
    -LocalPort 80 -Action Allow

# Remove a rule
Remove-NetFirewallRule -DisplayName "Allow HTTP"
```

---

## Remote management (PSRemoting)

```powershell
# Enable PowerShell remoting (run as admin on target machine)
Enable-PSRemoting -Force

# Connect to a remote machine
Enter-PSSession -ComputerName server01
Enter-PSSession -ComputerName 192.168.1.10 -Credential (Get-Credential)

# Run a command on a remote machine
Invoke-Command -ComputerName server01 -ScriptBlock {Get-Service}

# Run on multiple machines at once
Invoke-Command -ComputerName server01,server02,server03 -ScriptBlock {
    Get-Service -Name wuauserv | Select-Object Name, Status
}

# Copy a file to a remote machine
$Session = New-PSSession -ComputerName server01
Copy-Item -Path C:\script.ps1 -Destination C:\Temp\ -ToSession $Session
Remove-PSSession $Session
```

---

## Useful one-liners

```powershell
# Get public IP address
(Invoke-WebRequest -Uri "https://ifconfig.me").Content

# Check certificate expiry for a website
$Cert = [Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}
$Request = [Net.HttpWebRequest]::Create("https://weichertlabs.com")
$Request.GetResponse() | Out-Null
$Request.ServicePoint.Certificate.GetExpirationDateString()

# List installed software
Get-Package | Select-Object Name, Version | Sort-Object Name

# List installed Windows features
Get-WindowsOptionalFeature -Online | Where-Object {$_.State -eq "Enabled"}

# Generate a random password
Add-Type -AssemblyName System.Web
[System.Web.Security.Membership]::GeneratePassword(16, 4)

# Find which process is using a port
Get-NetTCPConnection -LocalPort 80 |
    Select-Object LocalPort, OwningProcess |
    ForEach-Object { Get-Process -Id $_.OwningProcess }

# Export running processes to CSV
Get-Process | Export-Csv -Path "$env:USERPROFILE\Desktop\processes.csv" -NoTypeInformation
```

---

## Related guides

- [PowerShell – Getting Started](/guides/operating-systems/windows/powershell-getting-started/) — start here if you're new to PowerShell
- [SSH to Windows – Remote Access](/guides/operating-systems/windows/ssh-windows-remote-access/) — connect from Mac or Linux
- [Windows Terminal – Setup and Tips](/guides/operating-systems/windows/windows-terminal-setup/) — best terminal experience on Windows
