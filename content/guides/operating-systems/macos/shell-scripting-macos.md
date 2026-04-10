---
title: Shell Scripting on macOS
description: A practical introduction to shell scripting on macOS — write your first script, understand variables and loops, and automate real tasks you actually do every day.
date: 2026-04-10
tags: [macos, bash, zsh, scripting, automation, terminal]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

Shell scripts are plain text files with commands in them — the same commands you type in the terminal, just saved so you can run them again. This guide covers the essentials and ends with practical scripts you can use immediately.

---

## Your first script

Create a file:

```bash
nano ~/scripts/hello.sh
```

Add:

```bash
#!/bin/zsh
echo "Hello from my first script!"
echo "Today is $(date)"
```

Save (`CTRL+O`, `CTRL+X`), make it executable, and run it:

```bash
chmod +x ~/scripts/hello.sh
~/scripts/hello.sh
```

The first line `#!/bin/zsh` tells the system which shell to use. Use `/bin/zsh` on macOS (the default shell since Catalina). Use `/bin/bash` if you need broader compatibility.

---

## Variables

```bash
#!/bin/zsh

# Assign a variable
NAME="Patrik"
AGE=35

# Use a variable with $
echo "Hello, $NAME"
echo "You are $AGE years old"

# Command substitution — store output of a command
TODAY=$(date +%Y-%m-%d)
echo "Today is $TODAY"

# Read from user input
echo "Enter your name:"
read USERNAME
echo "Hello, $USERNAME"
```

---

## Conditionals

```bash
#!/bin/zsh

FILE="config.yml"

# Check if file exists
if [ -f "$FILE" ]; then
    echo "File exists"
else
    echo "File not found"
fi

# Check if directory exists
if [ -d "/opt/docker" ]; then
    echo "Docker directory found"
fi

# Compare numbers
COUNT=5
if [ $COUNT -gt 3 ]; then
    echo "Count is greater than 3"
fi

# Compare strings
STATUS="running"
if [ "$STATUS" = "running" ]; then
    echo "Service is running"
elif [ "$STATUS" = "stopped" ]; then
    echo "Service is stopped"
else
    echo "Unknown status"
fi
```

**Common test operators:**

| Operator | Meaning |
|---|---|
| `-f file` | File exists |
| `-d dir` | Directory exists |
| `-z "$var"` | Variable is empty |
| `-n "$var"` | Variable is not empty |
| `$a -eq $b` | Numbers equal |
| `$a -gt $b` | Greater than |
| `$a -lt $b` | Less than |
| `"$a" = "$b"` | Strings equal |

---

## Loops

```bash
#!/bin/zsh

# Loop over a list
for FRUIT in apple banana cherry; do
    echo "Fruit: $FRUIT"
done

# Loop over files in a directory
for FILE in ~/Documents/*.pdf; do
    echo "Found PDF: $FILE"
done

# While loop
COUNT=1
while [ $COUNT -le 5 ]; do
    echo "Count: $COUNT"
    COUNT=$((COUNT + 1))
done

# Loop with a range
for i in {1..10}; do
    echo "Number $i"
done
```

---

## Functions in scripts

```bash
#!/bin/zsh

# Define a function
greet() {
    local NAME="$1"    # $1 is the first argument
    echo "Hello, $NAME!"
}

# Call the function
greet "Patrik"
greet "World"

# Function with return value (via echo)
get_date() {
    echo $(date +%Y-%m-%d)
}

TODAY=$(get_date)
echo "Today: $TODAY"
```

---

## Script arguments

```bash
#!/bin/zsh

# $0 = script name, $1 = first argument, $2 = second, etc.
# $# = number of arguments
# $@ = all arguments

echo "Script name: $0"
echo "First argument: $1"
echo "Number of arguments: $#"

# Check that an argument was provided
if [ $# -eq 0 ]; then
    echo "Usage: $0 <filename>"
    exit 1
fi

echo "Processing: $1"
```

Run it:

```bash
./myscript.sh myfile.txt
```

---

## Error handling

```bash
#!/bin/zsh

# Exit immediately if any command fails
set -e

# Exit on undefined variables
set -u

# Show commands as they run (useful for debugging)
set -x

# Check if a command succeeded
if ! cp source.txt destination.txt; then
    echo "Copy failed!"
    exit 1
fi

# Run cleanup on exit (even on error)
cleanup() {
    echo "Cleaning up..."
    rm -f /tmp/tempfile
}
trap cleanup EXIT
```

---

## Practical scripts

### Backup a folder with a timestamp

```bash
#!/bin/zsh
# backup.sh — backs up a folder to ~/Backups with a timestamp

SOURCE="${1:-$HOME/Documents}"
DEST="$HOME/Backups/$(basename $SOURCE)-$(date +%Y%m%d-%H%M%S).tar.gz"

mkdir -p "$HOME/Backups"

echo "Backing up $SOURCE..."
tar -czf "$DEST" "$SOURCE"
echo "Backup saved to: $DEST"
```

Usage:
```bash
./backup.sh ~/Documents
./backup.sh ~/projects/weichertlabs
```

---

### Clean up old files

```bash
#!/bin/zsh
# cleanup.sh — deletes files older than N days from a directory

DIR="${1:-.}"
DAYS="${2:-30}"

echo "Deleting files older than $DAYS days in $DIR"
find "$DIR" -type f -mtime +"$DAYS" -print -delete
echo "Done"
```

Usage:
```bash
./cleanup.sh ~/Downloads 30   # delete files older than 30 days
./cleanup.sh /tmp 7           # delete temp files older than 7 days
```

---

### Check if a service is running

```bash
#!/bin/zsh
# check-service.sh — checks if a Docker container is running

SERVICE="${1:-caddy}"

if docker ps --format '{{.Names}}' | grep -q "^${SERVICE}$"; then
    echo "✅ $SERVICE is running"
else
    echo "❌ $SERVICE is NOT running"
    echo "Starting $SERVICE..."
    cd /opt/docker/$SERVICE && docker compose up -d
fi
```

---

### Git pull all repos in a folder

```bash
#!/bin/zsh
# git-update-all.sh — runs git pull in all subdirectories

BASE_DIR="${1:-$HOME/projects}"

for DIR in "$BASE_DIR"/*/; do
    if [ -d "$DIR/.git" ]; then
        echo "Updating: $DIR"
        git -C "$DIR" pull --quiet
    fi
done

echo "All repos updated"
```

---

### Daily system summary

```bash
#!/bin/zsh
# daily-summary.sh — prints a quick system status summary

echo "============================="
echo "  System Summary — $(date)"
echo "============================="
echo ""
echo "💾 Disk usage:"
df -h / | tail -1 | awk '{print "  Used: "$3" / "$2" ("$5")"}'
echo ""
echo "🧠 Memory:"
vm_stat | grep "Pages free" | awk '{printf "  Free pages: %s\n", $3}'
echo ""
echo "🐳 Running containers:"
docker ps --format "  {{.Names}} ({{.Status}})" 2>/dev/null || echo "  Docker not running"
echo ""
echo "📡 Tailscale:"
tailscale status 2>/dev/null | head -3 | sed 's/^/  /'
echo ""
```

---

## Making scripts easy to run

Instead of typing the full path every time, put your scripts in a directory that's in your `PATH`:

```bash
# Create a scripts directory
mkdir -p ~/scripts

# Add it to your PATH in ~/.zshrc
echo 'export PATH="$HOME/scripts:$PATH"' >> ~/.zshrc
source ~/.zshrc

# Now you can run scripts by name from anywhere
backup.sh ~/Documents
daily-summary.sh
```

---

## Scheduling scripts with cron

Run a script automatically on a schedule:

```bash
crontab -e
```

Add:

```
# Run daily-summary.sh every day at 9am
0 9 * * * /Users/patrik/scripts/daily-summary.sh >> /tmp/daily-summary.log 2>&1

# Run cleanup.sh every Sunday at 2am
0 2 * * 0 /Users/patrik/scripts/cleanup.sh ~/Downloads 30
```

**Cron syntax:**
```
Minute Hour DayOfMonth Month DayOfWeek Command
  0      9      *         *      *      command
```

---

## Related guides

- [Essential macOS Terminal Commands](/guides/operating-systems/macos/essential-macos-terminal-commands/) — the commands used in scripts
- [macOS Terminal – Power User Tips](/guides/operating-systems/macos/terminal-power-user-macos/) — functions, pipes and text manipulation
- [Bash/Zsh Aliases on macOS](/guides/operating-systems/macos/aliases-macos/) — quick shortcuts alongside scripts
