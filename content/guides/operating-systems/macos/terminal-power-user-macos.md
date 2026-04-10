---
title: macOS Terminal – Power User Tips
description: Level up your macOS terminal skills — functions, text manipulation, process control, pipes, tmux, and tools that make you significantly faster.
date: 2026-04-10
tags: [macos, terminal, zsh, power-user, productivity]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

You know the basics. You can navigate the filesystem, install packages with Homebrew, and you have your aliases set up. This guide takes the next step — the tools and techniques that separate casual terminal users from people who live in it.

---

## Functions — more powerful than aliases

Aliases are fixed shortcuts. Functions accept arguments, making them far more flexible.

Add these to your `~/.zshrc`:

```bash
# Create a directory and cd into it immediately
mkcd() {
    mkdir -p "$1" && cd "$1"
}

# Find a file by name anywhere from current directory
ff() {
    find . -name "*$1*" 2>/dev/null
}

# Search for text recursively and show filename + line number
rg() {
    grep -rn "$1" . --include="*.${2:-*}"
}

# Extract any archive format
extract() {
    case "$1" in
        *.tar.gz|*.tgz)  tar xzf "$1" ;;
        *.tar.bz2)       tar xjf "$1" ;;
        *.tar.xz)        tar xJf "$1" ;;
        *.zip)           unzip "$1" ;;
        *.gz)            gunzip "$1" ;;
        *.rar)           unrar x "$1" ;;
        *.7z)            7z x "$1" ;;
        *)               echo "Unknown format: $1" ;;
    esac
}

# Quick HTTP server in current directory
serve() {
    local port="${1:-8000}"
    echo "Serving on http://localhost:$port"
    python3 -m http.server "$port"
}

# Show top 10 largest files in current directory
biggest() {
    du -sh * | sort -rh | head -10
}

# Kill whatever is running on a given port
killport() {
    lsof -ti ":$1" | xargs kill -9
}
```

Usage examples:

```bash
mkcd my-new-project    # creates folder and enters it
ff docker-compose      # finds files with docker-compose in the name
serve 3000             # starts HTTP server on port 3000
killport 8080          # kills whatever is on port 8080
extract archive.tar.gz # extracts any archive
```

---

## Text manipulation

### grep — search smarter

```bash
# Case-insensitive search
grep -i "error" logfile.txt

# Show line numbers
grep -n "error" logfile.txt

# Show 3 lines before and after each match
grep -C 3 "error" logfile.txt

# Count matches
grep -c "error" logfile.txt

# Invert — show lines that do NOT match
grep -v "debug" logfile.txt

# Search multiple files recursively
grep -r "TODO" ~/projects/ --include="*.py"
```

### sed — find and replace

```bash
# Replace first occurrence per line
sed 's/old/new/' file.txt

# Replace all occurrences per line
sed 's/old/new/g' file.txt

# Edit file in place (no backup)
sed -i '' 's/old/new/g' file.txt

# Delete lines matching a pattern
sed '/debug/d' file.txt

# Print only lines 5-10
sed -n '5,10p' file.txt
```

### awk — column manipulation

```bash
# Print the second column of a file
awk '{print $2}' file.txt

# Print lines where column 3 is greater than 100
awk '$3 > 100' file.txt

# Sum all values in column 1
awk '{sum += $1} END {print sum}' file.txt

# Use comma as delimiter (for CSV)
awk -F',' '{print $1, $3}' data.csv

# Print filename and line count for all .md files
awk 'END {print FILENAME, NR}' *.md
```

---

## find — actually useful

```bash
# Find files modified in the last 7 days
find . -mtime -7 -type f

# Find files larger than 100MB
find . -size +100M

# Find and delete all .DS_Store files
find . -name ".DS_Store" -delete

# Find empty directories
find . -type d -empty

# Find files and execute a command on each
find . -name "*.log" -exec rm {} \;

# Find files by permission
find . -perm 644 -type f
```

---

## Pipes and redirection

```bash
# Pipe output of one command into another
cat file.txt | grep "error" | sort | uniq

# Redirect output to a file (overwrite)
ls -la > output.txt

# Append to a file
echo "new line" >> output.txt

# Redirect stderr to a file
command 2> errors.txt

# Redirect both stdout and stderr
command > output.txt 2>&1

# Discard output entirely
command > /dev/null 2>&1

# Use output of a command as input to another
diff <(ls dir1/) <(ls dir2/)
```

---

## Process management

```bash
# Run a command in the background
long-command &

# List background jobs
jobs

# Bring background job to foreground
fg %1

# Send running command to background (CTRL+Z then bg)
bg %1

# Run command immune to terminal close
nohup long-command &

# Find process by name
ps aux | grep nginx

# Kill process by name
pkill nginx

# Kill process by port
lsof -ti :3000 | xargs kill -9
```

---

## curl — making HTTP requests

```bash
# GET request
curl https://api.example.com/data

# GET with headers shown
curl -I https://example.com

# POST with JSON body
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -d '{"name": "Patrik", "email": "test@example.com"}'

# Download a file
curl -O https://example.com/file.zip

# Follow redirects
curl -L https://example.com

# Silent mode (no progress bar)
curl -s https://api.example.com/data

# Check your public IP
curl -s ifconfig.me
```

### jq — parse JSON in the terminal

```bash
brew install jq

# Pretty print JSON
curl -s https://api.github.com/users/octocat | jq .

# Extract a specific field
curl -s https://api.github.com/users/octocat | jq .name

# Extract from an array
curl -s https://api.github.com/users/octocat/repos | jq '.[0].name'

# Filter array items
curl -s https://api.github.com/users/octocat/repos | jq '.[] | select(.language == "Ruby") | .name'
```

---

## tmux — never lose your work

tmux is a terminal multiplexer — it lets you split your terminal into panes, keep sessions alive when you disconnect from SSH, and switch between multiple workspaces.

```bash
brew install tmux
```

### Essential tmux commands

All tmux commands start with the prefix `CTRL+B`.

| Action | Keys |
|---|---|
| New session | `tmux new -s name` |
| Attach to session | `tmux attach -t name` |
| List sessions | `tmux ls` |
| Detach (keep running) | `CTRL+B d` |
| New window | `CTRL+B c` |
| Next window | `CTRL+B n` |
| Previous window | `CTRL+B p` |
| Split vertically | `CTRL+B %` |
| Split horizontally | `CTRL+B "` |
| Navigate panes | `CTRL+B arrow key` |
| Resize pane | `CTRL+B CTRL+arrow` |
| Kill pane | `CTRL+B x` |
| Scroll mode | `CTRL+B [` then arrow keys |

### Practical tmux workflow

```bash
# Start a named session for your project
tmux new -s weichertlabs

# Split into two panes — editor on left, terminal on right
CTRL+B %

# Detach — session keeps running even after closing the terminal
CTRL+B d

# Reattach later
tmux attach -t weichertlabs
```

This is especially useful when SSH'd into a remote server — your work continues even if the connection drops.

---

## Useful one-liners

```bash
# Show directory size sorted largest first
du -sh */ | sort -rh

# Count lines in all .md files
find . -name "*.md" | xargs wc -l

# Show last 50 commands from history
history | tail -50

# Search command history
history | grep docker

# Repeat last command with sudo
sudo !!

# Create a dated backup of a file
cp config.yml config.yml.$(date +%Y%m%d)

# Watch a command update every 2 seconds
watch -n 2 docker ps

# Show open network connections
lsof -i -n -P

# Generate a random password
openssl rand -base64 32

# Check certificate expiry for a domain
echo | openssl s_client -connect weichertlabs.com:443 2>/dev/null | openssl x509 -noout -dates
```

---

## Related guides

- [Essential macOS Terminal Commands](/guides/operating-systems/macos/essential-macos-terminal-commands/) — start here if you haven't already
- [Bash/Zsh Aliases on macOS](/guides/operating-systems/macos/aliases-macos/) — build your shortcut library
- [Shell Scripting on macOS](/guides/operating-systems/macos/shell-scripting-macos/) — put these tools to work in scripts
- [Oh My Zsh – Supercharge Your Terminal on macOS](/guides/operating-systems/macos/oh-my-zsh-macos/) — plugins that add power to zsh
