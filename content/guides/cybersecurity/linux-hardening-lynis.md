---
title: Linux System Hardening with Lynis
description: How to audit and harden a Linux server using Lynis — scan for vulnerabilities, understand the results, and fix the most common issues on Ubuntu and Debian.
date: 2026-04-10
tags: [linux, security, hardening, lynis, ubuntu, debian, sysadmin]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

Lynis is an open-source security auditing tool for Linux and Unix systems. It scans your system and generates a detailed report with findings, warnings, and suggestions — giving you a clear picture of your security posture and what to fix.

This guide covers installing Lynis, running your first audit, understanding the output, and fixing the most common findings on Ubuntu and Debian.

---

## Install Lynis

The version in the Ubuntu/Debian repositories is often outdated. Install directly from the Lynis project:

```bash
# Add the Lynis repository
curl -fsSL https://packages.cisofy.com/keys/cisofy-software-public.key | \
  sudo gpg --dearmor -o /usr/share/keyrings/cisofy-software-public.gpg

echo "deb [signed-by=/usr/share/keyrings/cisofy-software-public.gpg] \
  https://packages.cisofy.com/community/lynis/deb/ stable main" | \
  sudo tee /etc/apt/sources.list.d/cisofy-lynis.list

sudo apt update
sudo apt install -y lynis
```

Verify installation:

```bash
lynis show version
```

---

## Run a system audit

```bash
sudo lynis audit system
```

The scan takes 1-3 minutes. Lynis checks hundreds of items — SSH configuration, filesystem permissions, user accounts, kernel settings, installed software, and more.

---

## Understanding the output

The report is colour-coded in the terminal:

- **Green** — OK, no issues found
- **Yellow** — Suggestion or warning — worth looking at
- **Red** — Warning — should be fixed

At the end of the scan, look for the **Hardening Index**:

```
Hardening index : 58 [###########         ]
```

A fresh Ubuntu server typically scores 50-65. After hardening, aim for 75+.

The full report is saved to:
```
/var/log/lynis.log
/var/log/lynis-report.dat
```

---

## Common findings and how to fix them

### 1. SSH hardening

Lynis almost always flags SSH configuration. Edit `/etc/ssh/sshd_config`:

```bash
sudo nano /etc/ssh/sshd_config
```

Apply these settings:

```
# Disable root login
PermitRootLogin no

# Disable password authentication (use keys only)
PasswordAuthentication no

# Disable empty passwords
PermitEmptyPasswords no

# Use SSH protocol 2 only
Protocol 2

# Limit authentication attempts
MaxAuthTries 3

# Disable X11 forwarding if not needed
X11Forwarding no

# Disable unused authentication methods
UsePAM yes
ChallengeResponseAuthentication no

# Set login grace time
LoginGraceTime 30

# Only allow specific users (optional)
AllowUsers patrik
```

Restart SSH:

```bash
sudo systemctl restart sshd
```

{{< callout type="warning" >}}
Before disabling password authentication, make sure your SSH keys are set up and working. Test in a new terminal window before closing your existing session.
{{< /callout >}}

---

### 2. Kernel hardening (sysctl)

Lynis often suggests kernel parameter changes. Add them to a sysctl config file:

```bash
sudo nano /etc/sysctl.d/99-hardening.conf
```

Add:

```
# Disable IP forwarding (unless this is a router)
net.ipv4.ip_forward = 0

# Enable SYN flood protection
net.ipv4.tcp_syncookies = 1

# Disable ICMP redirects
net.ipv4.conf.all.accept_redirects = 0
net.ipv6.conf.all.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0

# Disable source routing
net.ipv4.conf.all.accept_source_route = 0
net.ipv6.conf.all.accept_source_route = 0

# Log suspicious packets
net.ipv4.conf.all.log_martians = 1

# Protect against IP spoofing
net.ipv4.conf.all.rp_filter = 1

# Disable IPv6 if not used
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1

# Restrict core dumps
fs.suid_dumpable = 0

# Protect against PTRACE (process tracing)
kernel.yama.ptrace_scope = 1

# Randomize memory layout (ASLR)
kernel.randomize_va_space = 2
```

Apply:

```bash
sudo sysctl -p /etc/sysctl.d/99-hardening.conf
```

---

### 3. Automatic security updates

```bash
sudo apt install -y unattended-upgrades

# Configure to auto-install security updates
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

Verify the configuration:

```bash
cat /etc/apt/apt.conf.d/50unattended-upgrades
```

Make sure the security lines are uncommented:

```
Unattended-Upgrade::Allowed-Origins {
    "${distro_id}:${distro_codename}-security";
};
```

---

### 4. Install and configure auditd

Lynis recommends audit logging for tracking system events:

```bash
sudo apt install -y auditd audispd-plugins

sudo systemctl enable auditd
sudo systemctl start auditd
```

Add basic audit rules:

```bash
sudo nano /etc/audit/rules.d/hardening.rules
```

```
# Log authentication events
-w /etc/passwd -p wa -k identity
-w /etc/shadow -p wa -k identity
-w /etc/group -p wa -k identity

# Log SSH configuration changes
-w /etc/ssh/sshd_config -p wa -k sshd

# Log sudo usage
-w /var/log/sudo.log -p wa -k sudo

# Log privilege escalation
-a always,exit -F arch=b64 -S setuid -k privilege_escalation
```

Reload rules:

```bash
sudo augenrules --load
```

---

### 5. File permissions

Lynis checks for world-writable files and files with incorrect permissions:

```bash
# Find world-writable files (excluding /proc and /sys)
find / -xdev -type f -perm -o+w 2>/dev/null | grep -v '^/proc\|^/sys'

# Find SUID files (should be a short list)
find / -xdev -perm -4000 -type f 2>/dev/null

# Fix common permission issues
sudo chmod 644 /etc/passwd
sudo chmod 640 /etc/shadow
sudo chmod 644 /etc/group
sudo chmod 700 /root
sudo chmod 700 /home/patrik/.ssh
sudo chmod 600 /home/patrik/.ssh/authorized_keys
```

---

### 6. Install fail2ban

Fail2ban monitors log files and automatically bans IPs with too many failed login attempts:

```bash
sudo apt install -y fail2ban

sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

Configure SSH protection:

```bash
sudo nano /etc/fail2ban/jail.local
```

```ini
[DEFAULT]
bantime = 1h
findtime = 10m
maxretry = 5

[sshd]
enabled = true
port = ssh
logpath = /var/log/auth.log
```

Restart fail2ban:

```bash
sudo systemctl restart fail2ban

# Check ban status
sudo fail2ban-client status sshd
```

---

### 7. Disable unused services

List all running services and disable what you don't need:

```bash
systemctl list-units --type=service --state=running

# Example: disable a service you don't use
sudo systemctl disable --now avahi-daemon
sudo systemctl disable --now cups
```

---

## Run Lynis again after hardening

After applying fixes, run the audit again to see your improved score:

```bash
sudo lynis audit system
```

Compare the new Hardening Index with your baseline. Each fix should improve the score.

---

## Automate monthly audits

```bash
sudo crontab -e
```

Add:

```
# Run Lynis audit monthly and save report
0 3 1 * * lynis audit system --quiet >> /var/log/lynis-monthly.log 2>&1
```

---

## Related guides

- [SSH Keys – The Right Way](/guides/operating-systems/linux/ssh-keys-linux/) — required before disabling password auth
- [UFW Firewall on Ubuntu and Debian](/guides/operating-systems/linux/ufw-firewall-linux/) — complementary firewall setup
- [Shell Scripting on Linux](/guides/operating-systems/linux/shell-scripting-linux/) — automate your hardening tasks
- [Lynis Documentation](https://cisofy.com/documentation/lynis/) — official docs
