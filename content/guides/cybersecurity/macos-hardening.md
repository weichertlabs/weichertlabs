---
title: macOS Security Hardening
description: How to harden a macOS system — FileVault, Firewall, Gatekeeper, privacy settings, and Lynis auditing for Mac. Practical security improvements for everyday users and IT professionals.
date: 2026-04-10
tags: [macos, security, hardening, filevault, firewall, privacy]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

macOS has strong security built in, but many of the most important protections are disabled by default or configured too loosely. This guide covers the practical steps to significantly improve your Mac's security posture.

---

## 1. Enable FileVault (disk encryption)

FileVault encrypts your entire disk. Without it, anyone with physical access to your Mac can read your files.

```
System Settings → Privacy & Security → FileVault → Turn On
```

Or via Terminal:

```bash
# Check status
sudo fdesetup status

# Enable FileVault
sudo fdesetup enable
```

{{< callout type="info" >}}
Save the recovery key somewhere safe — a password manager like Vaultwarden works well. Without it, you cannot recover your data if you forget your password.
{{< /callout >}}

---

## 2. Enable the built-in Firewall

macOS has a built-in application firewall that blocks unsolicited incoming connections:

```
System Settings → Network → Firewall → Turn On
```

Then click **Options** and enable:
- **Block all incoming connections** — if you don't run servers (very secure but aggressive)
- **Enable stealth mode** — makes your Mac invisible to network scans (recommended)

Via Terminal:

```bash
# Enable firewall
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setglobalstate on

# Enable stealth mode
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setstealthmode on

# Check status
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --getglobalstate
```

---

## 3. Gatekeeper and XProtect

Gatekeeper prevents unsigned or unverified apps from running. Ensure it's enabled:

```bash
# Check status
spctl --status

# Enable if disabled
sudo spctl --master-enable
```

Keep Gatekeeper set to **App Store and identified developers** at minimum:

```
System Settings → Privacy & Security → Allow apps downloaded from → App Store and identified developers
```

XProtect (Apple's built-in malware scanner) updates automatically in the background. No configuration needed.

---

## 4. Automatic security updates

Enable automatic security updates to ensure patches are applied promptly:

```
System Settings → General → Software Update → Automatic Updates → Enable All
```

Make sure these are checked:
- Check for updates
- Download new updates when available
- Install macOS updates
- Install application updates from the App Store
- **Install Security Responses and system files** — most important

Via Terminal:

```bash
# Enable automatic updates
sudo defaults write /Library/Preferences/com.apple.SoftwareUpdate AutomaticCheckEnabled -bool true
sudo defaults write /Library/Preferences/com.apple.SoftwareUpdate AutomaticDownload -bool true
sudo defaults write /Library/Preferences/com.apple.SoftwareUpdate CriticalUpdateInstall -bool true
```

---

## 5. Screen lock and password settings

```
System Settings → Lock Screen
```

Set:
- **Require password after screen saver begins:** Immediately (or 5 minutes at most)
- **Start screen saver when inactive:** 5 minutes

Strong password policy:

```
System Settings → Touch ID & Password → Change Password
```

Use a strong password — at least 12 characters with mixed case, numbers, and symbols.

---

## 6. Privacy settings — review app permissions

Audit which apps have access to sensitive data:

```
System Settings → Privacy & Security
```

Review and revoke access for apps that don't need it:
- **Location Services** — disable for apps that don't need your location
- **Camera** — check which apps have access
- **Microphone** — same
- **Contacts, Calendars, Reminders** — revoke for apps you don't trust
- **Full Disk Access** — this is powerful, only essential apps should have it
- **Accessibility** — review carefully, this allows controlling your Mac

---

## 7. Disable unnecessary sharing services

```
System Settings → General → Sharing
```

Disable everything you don't actively use:
- File Sharing
- Screen Sharing
- Remote Login (SSH)
- Remote Management
- AirPlay Receiver (if not needed)
- Bluetooth Sharing

Only enable what you actually need.

---

## 8. Secure Safari (or your browser)

**Safari settings:**
```
Safari → Settings → Privacy
```
- Enable **Prevent cross-site tracking**
- Enable **Hide IP address from trackers**
- Enable **Block all cookies** (may break some sites)

**For all browsers:**
- Use a password manager instead of the built-in one
- Enable HTTPS-only mode where available
- Review and remove extensions you don't actively use — each extension has access to your browsing

---

## 9. Check for listening network services

See what's listening on your network interfaces:

```bash
# Show all listening ports
sudo lsof -i -n -P | grep LISTEN

# Shorter view
netstat -an | grep LISTEN
```

Investigate anything unexpected and disable services you don't recognize.

---

## 10. Audit with Lynis

Lynis works on macOS and gives you a Linux-style security audit:

```bash
brew install lynis

# Run the audit
sudo lynis audit system
```

Review the suggestions in the output — many will be macOS-specific findings around permissions, configuration, and software.

---

## 11. Enable Lockdown Mode (high security)

For users who need maximum security (journalists, researchers, high-risk profiles):

```
System Settings → Privacy & Security → Lockdown Mode → Turn On
```

{{< callout type="warning" >}}
Lockdown Mode significantly restricts functionality — many apps and features will not work. It's designed for users under active threat, not everyday use.
{{< /callout >}}

---

## 12. Terminal-based security checks

```bash
# Check if SIP (System Integrity Protection) is enabled
csrutil status
# Should say: System Integrity Protection status: enabled.

# Check FileVault status
sudo fdesetup status

# Check firewall status
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --getglobalstate

# List apps with Full Disk Access
tccutil list SystemPolicyAllFiles

# Check for world-writable files in home directory
find ~ -perm -o+w -type f 2>/dev/null | grep -v Library

# List all login items
osascript -e 'tell application "System Events" to get the name of every login item'
```

---

## Security checklist

| Setting | Status |
|---|---|
| FileVault enabled | ✅ / ❌ |
| Firewall enabled | ✅ / ❌ |
| Stealth mode enabled | ✅ / ❌ |
| Auto security updates | ✅ / ❌ |
| Screen lock on sleep | ✅ / ❌ |
| Gatekeeper enabled | ✅ / ❌ |
| SIP enabled | ✅ / ❌ |
| Sharing services reviewed | ✅ / ❌ |
| App permissions audited | ✅ / ❌ |

---

## Related guides

- [Linux System Hardening with Lynis](/guides/cybersecurity/linux-hardening-lynis/) — same concepts for Linux servers
- [Windows Security Hardening](/guides/cybersecurity/windows-hardening/) — hardening for Windows systems
- [SSH Keys – The Right Way](/guides/operating-systems/linux/ssh-keys-linux/) — secure remote access
- [Lynis Documentation](https://cisofy.com/documentation/lynis/) — official docs
