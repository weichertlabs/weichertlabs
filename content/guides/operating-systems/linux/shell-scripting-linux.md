---
title: Shell Scripting on Linux
description: A practical introduction to shell scripting on Linux — variables, loops, conditionals, error handling, and real scripts for server administration on Ubuntu and Debian.
date: 2026-04-10
tags: [linux, bash, scripting, automation, sysadmin, ubuntu, debian]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

Shell scripts are plain text files with commands in them — the same commands you type in the terminal, saved so you can run them again. This guide covers the essentials with a focus on real server administration tasks.

---

## Your first script

```bash
nano ~/scripts/hello.sh
```

```bash
#!/bin/bash
echo "Hello from my first script!"
echo "Server: $(hostname)"
echo "Date: $(date)"
echo "Uptime: $(uptime -p)"
```

Make it executable and run:

```bash
chmod +x ~/scripts/hello.sh
~/scripts/hello.sh
```

The `#!/bin/bash` shebang tells the system to use bash. Always include it.

---

## Variables

```bash
#!/bin/bash

# Assign variables
SERVER_NAME="webserver-01"
PORT=8080

# Use with $
echo "Server: $SERVER_NAME on port $PORT"

# Command substitution
DISK_USAGE=$(df -h / | awk 'NR==2 {print $5}')
echo "Root disk usage: $DISK_USAGE"

# Default value if variable is empty
LOG_DIR="${LOG_DIR:-/var/log}"
echo "Log directory: $LOG_DIR"
```

---

## Conditionals

```bash
#!/bin/bash

SERVICE="nginx"

# Check if a service is running
if systemctl is-active --quiet "$SERVICE"; then
    echo "$SERVICE is running"
else
    echo "$SERVICE is NOT running"
    sudo systemctl start "$SERVICE"
fi

# Check if a file exists
CONFIG="/etc/nginx/nginx.conf"
if [ -f "$CONFIG" ]; then
    echo "Config found: $CONFIG"
fi

# Check if a directory exists
if [ ! -d "/opt/docker" ]; then
    mkdir -p /opt/docker
    echo "Created /opt/docker"
fi

# Compare numbers
FREE_DISK=$(df / | awk 'NR==2 {print $4}')
if [ "$FREE_DISK" -lt 1000000 ]; then
    echo "WARNING: Less than 1GB free on root partition"
fi
```

**Common test operators:**

| Operator | Meaning |
|---|---|
| `-f file` | File exists and is a regular file |
| `-d dir` | Directory exists |
| `-e path` | Path exists (file or directory) |
| `-z "$var"` | Variable is empty |
| `-n "$var"` | Variable is not empty |
| `$a -eq $b` | Numbers equal |
| `$a -gt $b` | Greater than |
| `$a -lt $b` | Less than |
| `"$a" = "$b"` | Strings equal |
| `"$a" != "$b"` | Strings not equal |

---

## Loops

```bash
#!/bin/bash

# Loop over a list
for SERVICE in nginx docker tailscaled; do
    if systemctl is-active --quiet "$SERVICE"; then
        echo "✅ $SERVICE"
    else
        echo "❌ $SERVICE"
    fi
done

# Loop over files
for FILE in /opt/docker/*/compose.yml; do
    DIR=$(dirname "$FILE")
    echo "Found stack: $DIR"
done

# While loop — retry until success
RETRIES=0
while ! ping -c 1 google.com > /dev/null 2>&1; do
    RETRIES=$((RETRIES + 1))
    echo "Network not ready, retrying... ($RETRIES)"
    sleep 5
    if [ "$RETRIES" -ge 10 ]; then
        echo "Network check failed after 10 retries"
        exit 1
    fi
done
echo "Network is up"

# Loop with a range
for i in {1..5}; do
    echo "Attempt $i"
done
```

---

## Functions

```bash
#!/bin/bash

# Simple function
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1"
}

# Function with return via echo
get_disk_usage() {
    df -h "$1" | awk 'NR==2 {print $5}' | tr -d '%'
}

# Function with local variables
check_service() {
    local SERVICE="$1"
    if systemctl is-active --quiet "$SERVICE"; then
        log "$SERVICE is running"
        return 0
    else
        log "$SERVICE is NOT running"
        return 1
    fi
}

# Usage
log "Script started"
check_service nginx
USAGE=$(get_disk_usage /)
log "Disk usage: ${USAGE}%"
```

---

## Script arguments

```bash
#!/bin/bash

# Check required arguments
if [ $# -lt 1 ]; then
    echo "Usage: $0 <service> [action]"
    echo "Example: $0 nginx restart"
    exit 1
fi

SERVICE="$1"
ACTION="${2:-status}"

case "$ACTION" in
    start)    sudo systemctl start "$SERVICE" ;;
    stop)     sudo systemctl stop "$SERVICE" ;;
    restart)  sudo systemctl restart "$SERVICE" ;;
    status)   systemctl status "$SERVICE" ;;
    *)        echo "Unknown action: $ACTION"; exit 1 ;;
esac
```

---

## Error handling

```bash
#!/bin/bash

# Exit on any error
set -e

# Exit on undefined variables
set -u

# Catch errors in pipelines
set -o pipefail

# Log function
log() { echo "[$(date '+%H:%M:%S')] $1"; }
error() { echo "[ERROR] $1" >&2; exit 1; }

# Cleanup on exit
cleanup() {
    log "Cleaning up..."
    rm -f /tmp/deploy-lock
}
trap cleanup EXIT

# Check we're running as root
if [ "$EUID" -ne 0 ]; then
    error "This script must be run as root"
fi

log "Starting deployment..."
```

---

## Practical server scripts

### System health check

```bash
#!/bin/bash
# health-check.sh — quick server health summary

log() { echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1"; }

echo "========================================"
echo "  Health Check — $(hostname)"
echo "  $(date)"
echo "========================================"

# Disk usage
DISK=$(df -h / | awk 'NR==2 {print $5}' | tr -d '%')
if [ "$DISK" -gt 85 ]; then
    echo "⚠️  Disk: ${DISK}% (WARNING)"
else
    echo "✅ Disk: ${DISK}%"
fi

# Memory
MEM_FREE=$(free -m | awk 'NR==2 {print $4}')
echo "💾 Free memory: ${MEM_FREE}MB"

# Load average
LOAD=$(uptime | awk -F'load average:' '{print $2}' | xargs)
echo "⚡ Load average: $LOAD"

# Services
echo ""
echo "Services:"
for SERVICE in nginx docker tailscaled ssh; do
    if systemctl is-active --quiet "$SERVICE" 2>/dev/null; then
        echo "  ✅ $SERVICE"
    else
        echo "  ❌ $SERVICE"
    fi
done

# Docker containers
if command -v docker &>/dev/null; then
    echo ""
    echo "Docker containers:"
    docker ps --format "  {{.Names}} — {{.Status}}" 2>/dev/null
fi
```

---

### Backup a directory

```bash
#!/bin/bash
# backup.sh — tar backup with timestamp and optional remote copy

SOURCE="${1:?Usage: $0 <source-dir> [dest-dir]}"
DEST_DIR="${2:-/mnt/backups}"
TIMESTAMP=$(date +%Y%m%d-%H%M%S)
FILENAME="$(basename "$SOURCE")-${TIMESTAMP}.tar.gz"
DEST="${DEST_DIR}/${FILENAME}"

mkdir -p "$DEST_DIR"

echo "Backing up $SOURCE → $DEST"
tar -czf "$DEST" "$SOURCE"

SIZE=$(du -sh "$DEST" | cut -f1)
echo "Done. Size: $SIZE"

# Keep only last 7 backups
ls -t "${DEST_DIR}/$(basename "$SOURCE")-"*.tar.gz 2>/dev/null | tail -n +8 | xargs rm -f
echo "Old backups cleaned up"
```

---

### Docker stack update

```bash
#!/bin/bash
# update-stacks.sh — pull and restart all Docker Compose stacks

DOCKER_BASE="/opt/docker"
LOG_FILE="/var/log/stack-updates.log"

log() { echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"; }

log "Starting stack updates..."

for COMPOSE_FILE in "${DOCKER_BASE}"/*/compose.yml; do
    STACK_DIR=$(dirname "$COMPOSE_FILE")
    STACK_NAME=$(basename "$STACK_DIR")

    log "Updating: $STACK_NAME"
    cd "$STACK_DIR"

    if docker compose pull --quiet 2>&1 | tee -a "$LOG_FILE"; then
        docker compose up -d --quiet-pull 2>&1 | tee -a "$LOG_FILE"
        log "$STACK_NAME updated successfully"
    else
        log "ERROR: Failed to pull $STACK_NAME"
    fi
done

log "All stacks updated"
```

---

### Log monitor and alert

```bash
#!/bin/bash
# log-monitor.sh — watch a log file and alert on keywords

LOG_FILE="${1:-/var/log/syslog}"
KEYWORDS="error|critical|failed|panic"
ALERT_FILE="/tmp/alerts.log"

echo "Monitoring $LOG_FILE for: $KEYWORDS"
echo "Alerts logged to: $ALERT_FILE"

tail -f "$LOG_FILE" | grep --line-buffered -iE "$KEYWORDS" | while read -r LINE; do
    TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S')
    echo "[$TIMESTAMP] ALERT: $LINE" | tee -a "$ALERT_FILE"
done
```

---

## Scheduling with cron

```bash
crontab -e
```

```
# Health check every hour
0 * * * * /home/patrik/scripts/health-check.sh >> /var/log/health.log 2>&1

# Backup Docker configs every night at 2am
0 2 * * * /home/patrik/scripts/backup.sh /opt/docker /mnt/backups

# Update all Docker stacks every Sunday at 3am
0 3 * * 0 /home/patrik/scripts/update-stacks.sh

# Clean up old logs weekly
0 4 * * 0 find /var/log -name "*.log" -mtime +30 -delete
```

**Cron syntax:**
```
Minute Hour DayOfMonth Month DayOfWeek Command
  0      2       *         *      *      command
```

Use [crontab.guru](https://crontab.guru) to build and test cron expressions.

---

## Making scripts easy to run

```bash
# Create a scripts directory
mkdir -p ~/scripts

# Add to PATH in ~/.bashrc
echo 'export PATH="$HOME/scripts:$PATH"' >> ~/.bashrc
source ~/.bashrc

# Now run scripts from anywhere
health-check.sh
backup.sh /opt/docker
```

---

## Related guides

- [Essential Linux Commands](/guides/operating-systems/linux/essential-linux-commands/) — the commands used in scripts
- [Linux Terminal – Power User Tips](/guides/operating-systems/linux/terminal-power-user-linux/) — grep, awk, sed, and more
- [Bash Aliases on Linux](/guides/operating-systems/linux/aliases-linux/) — quick shortcuts alongside scripts
- [SSH Keys – The Right Way](/guides/operating-systems/linux/ssh-keys-linux/) — secure remote access for automated scripts
