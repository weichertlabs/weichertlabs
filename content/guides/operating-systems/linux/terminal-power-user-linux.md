---
title: Linux Terminal – Power User Tips
description: Level up your Linux terminal skills — text manipulation, process control, systemd, journalctl, networking tools, tmux, and one-liners for server administration.
date: 2026-04-10
tags: [linux, terminal, bash, power-user, sysadmin, ubuntu, debian]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

You know the basics — navigation, SSH, UFW, aliases. This guide covers the tools and techniques that make Linux server administration significantly faster and more effective.

---

## Functions — more powerful than aliases

Add these to your `~/.bashrc`:

```bash
# Create a directory and cd into it
mkcd() {
    mkdir -p "$1" && cd "$1"
}

# Find a file by name from current directory
ff() {
    find . -name "*$1*" 2>/dev/null
}

# Search text recursively with line numbers
search() {
    grep -rn "$1" . 2>/dev/null
}

# Extract any archive format
extract() {
    case "$1" in
        *.tar.gz|*.tgz)  tar xzf "$1" ;;
        *.tar.bz2)       tar xjf "$1" ;;
        *.tar.xz)        tar xJf "$1" ;;
        *.zip)           unzip "$1" ;;
        *.gz)            gunzip "$1" ;;
        *)               echo "Unknown format: $1" ;;
    esac
}

# Kill whatever is running on a given port
killport() {
    fuser -k "${1}/tcp"
}

# Show top 10 largest items in current directory
biggest() {
    du -sh * | sort -rh | head -10
}

# Quick HTTP server in current directory
serve() {
    local port="${1:-8000}"
    echo "Serving on http://$(hostname -I | awk '{print $1}'):$port"
    python3 -m http.server "$port"
}
```

Reload:
```bash
source ~/.bashrc
```

---

## Text manipulation

### grep — search smarter

```bash
# Case-insensitive search
grep -i "error" /var/log/syslog

# Show line numbers
grep -n "failed" /var/log/auth.log

# Show 3 lines of context around each match
grep -C 3 "error" logfile.txt

# Count matches
grep -c "error" logfile.txt

# Invert — lines that do NOT match
grep -v "debug" logfile.txt

# Search recursively in a directory
grep -r "TODO" /opt/docker/ --include="*.yml"

# Extended regex
grep -E "error|warning|critical" /var/log/syslog
```

### sed — find and replace

```bash
# Replace all occurrences in a file
sed 's/old/new/g' file.txt

# Edit file in place
sed -i 's/old/new/g' file.txt

# Delete lines matching a pattern
sed -i '/debug/d' file.txt

# Print only lines 10-20
sed -n '10,20p' file.txt

# Remove blank lines
sed -i '/^$/d' file.txt
```

### awk — column processing

```bash
# Print second column
awk '{print $2}' file.txt

# Use custom delimiter (colon for /etc/passwd)
awk -F':' '{print $1}' /etc/passwd

# Print lines where column 3 > 1000
awk -F':' '$3 > 1000' /etc/passwd

# Sum column 1
awk '{sum += $1} END {print sum}' numbers.txt

# Print last column
awk '{print $NF}' file.txt
```

---

## find — the real way

```bash
# Find files modified in the last 24 hours
find /var/log -mtime -1 -type f

# Find files larger than 100MB
find / -size +100M -type f 2>/dev/null

# Find and delete all .log files older than 30 days
find /var/log -name "*.log" -mtime +30 -delete

# Find SUID files (security check)
find / -perm -4000 -type f 2>/dev/null

# Find world-writable files
find / -perm -o+w -type f 2>/dev/null

# Find and execute a command on each result
find /opt/docker -name "compose.yml" -exec echo "Found: {}" \;
```

---

## systemd and services

```bash
# Check service status
systemctl status nginx

# Start / stop / restart
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx

# Enable on boot / disable
sudo systemctl enable nginx
sudo systemctl disable nginx

# List all running services
systemctl list-units --type=service --state=running

# List failed services
systemctl list-units --state=failed

# Show service dependencies
systemctl list-dependencies nginx

# Edit service file override
sudo systemctl edit nginx

# Reload systemd after editing unit files
sudo systemctl daemon-reload
```

---

## journalctl — reading logs properly

```bash
# Follow live logs (like tail -f but better)
journalctl -f

# Follow logs for a specific service
journalctl -u nginx -f

# Show logs since last boot
journalctl -b

# Show logs from the previous boot
journalctl -b -1

# Show logs from the last hour
journalctl --since "1 hour ago"

# Show logs between timestamps
journalctl --since "2026-04-10 08:00" --until "2026-04-10 09:00"

# Show only errors and above
journalctl -p err

# Show kernel messages
journalctl -k

# Show logs for a specific PID
journalctl _PID=1234

# Clear old logs (keep last 500MB)
sudo journalctl --vacuum-size=500M
```

---

## Networking tools

```bash
# Show all IP addresses and interfaces
ip a

# Show routing table
ip r

# Show listening ports and which process
ss -tulnp

# Show established connections
ss -tnp state established

# Test connectivity
ping -c 4 google.com

# Trace route
traceroute google.com

# DNS lookup
dig weichertlabs.com
nslookup weichertlabs.com

# Check if a port is open on a remote host
nc -zv 192.168.1.10 22
nc -zv 192.168.1.10 80

# Download a file
wget https://example.com/file.tar.gz
curl -O https://example.com/file.tar.gz

# Check public IP
curl -s ifconfig.me

# Scan local network for hosts
nmap -sn 192.168.1.0/24
```

---

## Disk and storage

```bash
# Disk usage overview
df -h

# Size of a specific directory
du -sh /var/log/

# Largest directories under /
du -sh /* 2>/dev/null | sort -rh | head -10

# List block devices
lsblk

# Show disk I/O stats live
iostat -x 2

# Check filesystem for errors (unmounted)
sudo fsck /dev/sdb1

# Show inode usage
df -i
```

---

## Process management

```bash
# Interactive process viewer
htop

# Show processes using most CPU
ps aux --sort=-%cpu | head -10

# Show processes using most memory
ps aux --sort=-%mem | head -10

# Find a process by name
pgrep nginx
pidof nginx

# Kill by name
pkill nginx

# Kill by PID
kill -9 1234

# Run command immune to terminal close
nohup long-command > output.log 2>&1 &

# Show open files by a process
lsof -p 1234

# Show what files a process has open
lsof -u username
```

---

## tmux — essential for servers

tmux keeps your sessions alive when you disconnect from SSH — indispensable for long-running tasks.

```bash
sudo apt install -y tmux
```

### Key commands

| Action | Keys |
|---|---|
| New session | `tmux new -s name` |
| Detach (keep running) | `CTRL+B d` |
| Reattach | `tmux attach -t name` |
| List sessions | `tmux ls` |
| New window | `CTRL+B c` |
| Split vertically | `CTRL+B %` |
| Split horizontally | `CTRL+B "` |
| Navigate panes | `CTRL+B arrow` |
| Scroll mode | `CTRL+B [` |
| Kill pane | `CTRL+B x` |

### Practical server workflow

```bash
# Start a named session before running anything long
tmux new -s deploy

# Run your command
./deploy.sh

# Detach — it keeps running even if SSH drops
CTRL+B d

# Check back later
tmux attach -t deploy
```

---

## curl and APIs

```bash
# GET request
curl https://api.example.com/data

# GET with JSON output formatted
curl -s https://api.example.com/data | jq .

# POST with JSON
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -d '{"name": "Patrik"}'

# With authentication header
curl -H "Authorization: Bearer your-token" https://api.example.com/data

# Follow redirects, show response code
curl -L -o /dev/null -s -w "%{http_code}" https://weichertlabs.com

# Download with progress bar
curl -# -O https://example.com/large-file.tar.gz
```

---

## Useful server one-liners

```bash
# Show last 10 failed SSH login attempts
grep "Failed password" /var/log/auth.log | tail -10

# Show who is logged in
w

# Show last logins
last | head -20

# Check open ports
ss -tulnp

# Generate a secure password
openssl rand -base64 32

# Show certificate expiry for a domain
echo | openssl s_client -connect weichertlabs.com:443 2>/dev/null \
  | openssl x509 -noout -dates

# Watch Docker containers live
watch -n 2 docker ps

# Tail multiple log files at once
tail -f /var/log/syslog /var/log/auth.log

# Count unique IPs in a log file
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -20

# Show system uptime and load
uptime

# Show memory usage in MB
free -m

# Benchmark disk write speed
dd if=/dev/zero of=/tmp/test bs=1M count=1024 oflag=dsync
```

---

## Related guides

- [Essential Linux Commands](/guides/operating-systems/linux/essential-linux-commands/) — start here first
- [Bash Aliases on Linux](/guides/operating-systems/linux/aliases-linux/) — build your shortcut library
- [Shell Scripting on Linux](/guides/operating-systems/linux/shell-scripting-linux/) — automate everything
- [SSH Keys – The Right Way](/guides/operating-systems/linux/ssh-keys-linux/) — secure your server access
- [UFW Firewall on Ubuntu and Debian](/guides/operating-systems/linux/ufw-firewall-linux/) — lock down your server
