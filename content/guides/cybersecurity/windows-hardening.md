---
title: Windows Security Hardening
description: How to harden a Windows 10/11 system — BitLocker, Windows Defender, attack surface reduction, local security policy, and PowerShell-based hardening for workstations and servers.
date: 2026-04-10
tags: [windows, security, hardening, bitlocker, defender, powershell, sysadmin]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

Windows has a solid security foundation, but the default configuration leaves many protections disabled or weakened. This guide covers the practical steps to significantly improve Windows security — for both workstations and servers.

---

## 1. Enable BitLocker (disk encryption)

BitLocker encrypts your drive and protects data if the machine is stolen or tampered with.

**Via Settings (Windows 11):**
```
Settings → Privacy & Security → Device Encryption → Turn On
```

**Via PowerShell (requires admin):**

```powershell
# Check BitLocker status
Get-BitLockerVolume

# Enable BitLocker on C: drive
Enable-BitLocker -MountPoint "C:" `
    -EncryptionMethod XtsAes256 `
    -UsedSpaceOnly `
    -RecoveryPasswordProtector

# Save the recovery key
(Get-BitLockerVolume -MountPoint "C:").KeyProtector |
    Where-Object {$_.KeyProtectorType -eq "RecoveryPassword"} |
    Select-Object RecoveryPassword
```

{{< callout type="info" >}}
Save the recovery key somewhere secure — a password manager is ideal. Without it, you cannot recover data if something goes wrong.
{{< /callout >}}

---

## 2. Windows Defender — verify and configure

Windows Defender is built-in and good. Make sure it's fully enabled and configured:

```powershell
# Check status
Get-MpComputerStatus | Select-Object AMServiceEnabled, AntispywareEnabled, AntivirusEnabled, RealTimeProtectionEnabled

# Enable real-time protection if disabled
Set-MpPreference -DisableRealtimeMonitoring $false

# Enable cloud protection
Set-MpPreference -MAPSReporting Advanced
Set-MpPreference -SubmitSamplesConsent SendAllSamples

# Enable automatic sample submission
Set-MpPreference -SubmitSamplesConsent 1

# Run a quick scan
Start-MpScan -ScanType QuickScan

# Update virus definitions
Update-MpSignature
```

---

## 3. Attack Surface Reduction (ASR) rules

ASR rules block specific attack techniques commonly used by malware:

```powershell
# Enable key ASR rules (requires Windows Defender in active mode)
# Block Office apps from creating child processes
Add-MpPreference -AttackSurfaceReductionRules_Ids "D4F940AB-401B-4EFC-AADC-AD5F3C50688A" `
    -AttackSurfaceReductionRules_Actions Enabled

# Block credential stealing from Windows local security authority
Add-MpPreference -AttackSurfaceReductionRules_Ids "9e6c4e1f-7d60-472f-ba1a-a39ef669e4b3" `
    -AttackSurfaceReductionRules_Actions Enabled

# Block executable content from email and webmail
Add-MpPreference -AttackSurfaceReductionRules_Ids "BE9BA2D9-53EA-4CDC-84E5-9B1EEEE46550" `
    -AttackSurfaceReductionRules_Actions Enabled

# Block Office apps from creating executable content
Add-MpPreference -AttackSurfaceReductionRules_Ids "3B576869-A4EC-4529-8536-B80A7769E899" `
    -AttackSurfaceReductionRules_Actions Enabled

# Check current ASR rules
Get-MpPreference | Select-Object -ExpandProperty AttackSurfaceReductionRules_Ids
```

---

## 4. Windows Firewall

Windows Firewall is enabled by default. Verify and configure it:

```powershell
# Check firewall status for all profiles
Get-NetFirewallProfile | Select-Object Name, Enabled, DefaultInboundAction, DefaultOutboundAction

# Enable all profiles if any are disabled
Set-NetFirewallProfile -All -Enabled True

# Set default to block inbound, allow outbound
Set-NetFirewallProfile -All -DefaultInboundAction Block -DefaultOutboundAction Allow

# List all enabled inbound rules
Get-NetFirewallRule -Direction Inbound -Enabled True |
    Select-Object DisplayName, Profile, Action |
    Sort-Object DisplayName

# Disable a rule you don't need (example: Remote Desktop)
Disable-NetFirewallRule -DisplayName "Remote Desktop - User Mode (TCP-In)"
```

---

## 5. User account control (UAC)

UAC prompts for confirmation before elevated actions. Set it to maximum:

```
Control Panel → User Accounts → Change User Account Control settings → Always notify
```

Via registry:

```powershell
# Set UAC to always notify (highest level)
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" `
    -Name "ConsentPromptBehaviorAdmin" -Value 2

Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" `
    -Name "PromptOnSecureDesktop" -Value 1
```

---

## 6. Disable unnecessary features and services

```powershell
# List installed optional features
Get-WindowsOptionalFeature -Online | Where-Object {$_.State -eq "Enabled"} |
    Select-Object FeatureName

# Disable features you don't need
# Example: disable Telnet client
Disable-WindowsOptionalFeature -Online -FeatureName "TelnetClient" -NoRestart

# Example: disable SMB v1 (old, insecure)
Disable-WindowsOptionalFeature -Online -FeatureName "SMB1Protocol" -NoRestart

# Disable PowerShell v2 (older, less secure)
Disable-WindowsOptionalFeature -Online -FeatureName "MicrosoftWindowsPowerShellV2Root" -NoRestart
```

Disable unnecessary services:

```powershell
# List running services
Get-Service | Where-Object {$_.Status -eq "Running"} | Select-Object Name, DisplayName

# Disable a service (example: Remote Registry)
Set-Service -Name "RemoteRegistry" -StartupType Disabled
Stop-Service -Name "RemoteRegistry"

# Common services to consider disabling if not needed:
# RemoteRegistry - allows remote registry access
# Spooler - print spooler (if no printer)
# Fax - fax service
# XblAuthManager, XblGameSave, XboxNetApiSvc - Xbox services on non-gaming machines
```

---

## 7. Audit and logging

Enable comprehensive Windows event logging:

```powershell
# Enable audit logging for logon events
auditpol /set /subcategory:"Logon" /success:enable /failure:enable

# Enable audit for account management
auditpol /set /subcategory:"User Account Management" /success:enable /failure:enable

# Enable audit for privilege use
auditpol /set /subcategory:"Sensitive Privilege Use" /success:enable /failure:enable

# Enable object access auditing
auditpol /set /subcategory:"File System" /success:enable /failure:enable

# View current audit policy
auditpol /get /category:*
```

---

## 8. Secure PowerShell

Restrict PowerShell execution and enable logging:

```powershell
# Set execution policy to require signed scripts (for all users)
Set-ExecutionPolicy AllSigned -Scope LocalMachine

# Enable PowerShell script block logging (logs all script activity)
$Path = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging"
New-Item -Path $Path -Force
Set-ItemProperty -Path $Path -Name "EnableScriptBlockLogging" -Value 1

# Enable PowerShell transcription (full session logging)
$Path = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\Transcription"
New-Item -Path $Path -Force
Set-ItemProperty -Path $Path -Name "EnableTranscripting" -Value 1
Set-ItemProperty -Path $Path -Name "OutputDirectory" -Value "C:\PSLogs"
New-Item -ItemType Directory -Path "C:\PSLogs" -Force
```

---

## 9. Disable SMB signing bypass and guest access

```powershell
# Require SMB signing
Set-SmbServerConfiguration -RequireSecuritySignature $true -Force
Set-SmbClientConfiguration -RequireSecuritySignature $true -Force

# Disable guest access to SMB shares
Set-SmbServerConfiguration -EnableSMBQUICServer $false -Force

# Disable NetBIOS over TCP (reduces attack surface)
$Adapters = Get-WmiObject Win32_NetworkAdapterConfiguration
$Adapters | ForEach-Object { $_.SetTcpipNetbios(2) }
```

---

## 10. Security checklist (PowerShell)

Run this to get a quick security overview:

```powershell
Write-Host "=== Windows Security Check ===" -ForegroundColor Cyan
Write-Host ""

# BitLocker
$BL = Get-BitLockerVolume -MountPoint "C:" 2>$null
if ($BL.ProtectionStatus -eq "On") {
    Write-Host "✅ BitLocker: Enabled" -ForegroundColor Green
} else {
    Write-Host "❌ BitLocker: NOT enabled" -ForegroundColor Red
}

# Windows Defender
$WD = Get-MpComputerStatus
if ($WD.RealTimeProtectionEnabled) {
    Write-Host "✅ Windows Defender: Real-time protection on" -ForegroundColor Green
} else {
    Write-Host "❌ Windows Defender: Real-time protection OFF" -ForegroundColor Red
}

# Firewall
$FW = Get-NetFirewallProfile
$FW | ForEach-Object {
    if ($_.Enabled) {
        Write-Host "✅ Firewall ($($_.Name)): Enabled" -ForegroundColor Green
    } else {
        Write-Host "❌ Firewall ($($_.Name)): DISABLED" -ForegroundColor Red
    }
}

# Windows Update
$WU = Get-HotFix | Sort-Object InstalledOn -Descending | Select-Object -First 1
Write-Host "📦 Last update: $($WU.InstalledOn)" -ForegroundColor Yellow

# SMBv1
$SMB1 = Get-WindowsOptionalFeature -Online -FeatureName SMB1Protocol
if ($SMB1.State -eq "Disabled") {
    Write-Host "✅ SMBv1: Disabled" -ForegroundColor Green
} else {
    Write-Host "❌ SMBv1: ENABLED (disable this)" -ForegroundColor Red
}

Write-Host ""
Write-Host "=== Check complete ===" -ForegroundColor Cyan
```

---

## 11. Microsoft Security Compliance Toolkit

For organizations or advanced users, Microsoft provides security baselines — pre-configured Group Policy settings recommended by Microsoft:

Download from: [Microsoft Security Compliance Toolkit](https://www.microsoft.com/en-us/download/details.aspx?id=55319)

The toolkit includes baselines for Windows 10, Windows 11, and Windows Server — ready to import into Group Policy.

---

## Related guides

- [Linux System Hardening with Lynis](/guides/cybersecurity/linux-hardening-lynis/) — hardening for Linux servers
- [macOS Security Hardening](/guides/cybersecurity/macos-hardening/) — hardening for macOS
- [PowerShell – Practical Commands for IT](/guides/operating-systems/windows/powershell-practical-commands/) — PowerShell for Windows administration
- [SSH to Windows – Remote Access](/guides/operating-systems/windows/ssh-windows-remote-access/) — secure remote access
