---
title: Essential macOS Terminal Commands
description: A practical reference for macOS terminal commands — navigation, files, Homebrew, and macOS-specific tools you will actually use every day.
date: 2026-03-27
tags: [macos, terminal, commands, zsh, reference]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

macOS comes with a powerful Unix-based terminal that most users never fully explore. This guide covers the commands you will actually use day to day — from basic navigation to macOS-specific tools that make life easier.

macOS uses **zsh** as the default shell (since macOS Catalina). Most commands here work on Linux too, but some are macOS-specific.

---

## Navigation

| Command | What it does |
|---|---|
| `pwd` | Print current directory |
| `ls` | List files and folders |
| `ls -la` | List all files with details and hidden files |
| `cd /path/to/folder` | Change directory |
| `cd ..` | Go up one level |
| `cd ~` | Go to home directory |
| `cd -` | Go back to previous directory |

```bash
# List everything in your Documents folder
ls -la ~/Documents/
```

---

## Files and Directories

| Command | What it does |
|---|---|
| `mkdir foldername` | Create a directory |
| `mkdir -p a/b/c` | Create nested directories |
| `touch file.txt` | Create an empty file |
| `cp file.txt ~/destination/` | Copy a file |
| `cp -r folder/ ~/destination/` | Copy a folder recursively |
| `mv file.txt ~/destination/` | Move or rename a file |
| `rm file.txt` | Delete a file |
| `rm -rf folder/` | Delete a folder and all contents |
| `cat file.txt` | Print file contents |
| `less file.txt` | Scroll through file contents |
| `head -n 20 file.txt` | Show first 20 lines |
| `tail -f file.txt` | Follow file in real time |

{{< callout type="warning" >}}
`rm -rf` permanently deletes files with no Trash. There is no undo. Always double-check your path.
{{< /callout >}}

---

## macOS-Specific Commands

These are unique to macOS and some of the most useful ones to know:

| Command | What it does |
|---|---|
| `open .` | Open current folder in Finder |
| `open file.txt` | Open a file with its default app |
| `open -a "Visual Studio Code" .` | Open folder in a specific app |
| `pbcopy < file.txt` | Copy file contents to clipboard |
| `echo "hello" \| pbcopy` | Copy text to clipboard |
| `pbpaste` | Paste clipboard contents to terminal |
| `say "hello"` | Text to speech |
| `caffeinate` | Prevent Mac from sleeping |
| `caffeinate -t 3600` | Prevent sleep for 1 hour |
| `mdfind "filename"` | Spotlight search from terminal |
| `screencapture screenshot.png` | Take a screenshot |
| `diskutil list` | List all disks and partitions |

```bash
# Copy a command's output directly to clipboard
cat ~/.ssh/id_ed25519.pub | pbcopy

# Open current folder in Finder
open .

# Keep Mac awake while running a long script
caffeinate ./long-script.sh
```

---

## Searching

| Command | What it does |
|---|---|
| `grep "text" file.txt` | Search for text in a file |
| `grep -r "text" /path/` | Search recursively |
| `grep -i "text" file.txt` | Case-insensitive search |
| `find ~ -name "*.log"` | Find files by name |
| `mdfind "filename"` | Spotlight search (faster for indexed files) |

---

## System Information

| Command | What it does |
|---|---|
| `sw_vers` | macOS version |
| `uname -a` | Kernel and system info |
| `hostname` | Show hostname |
| `uptime` | How long the Mac has been running |
| `whoami` | Current user |
| `df -h` | Disk usage |
| `du -sh ~/folder/` | Size of a specific folder |
| `top` | Real-time process monitor |
| `vm_stat` | Memory usage statistics |
| `system_profiler SPHardwareDataType` | Detailed hardware info |

```bash
# Check macOS version
sw_vers

# See what's eating your disk space
du -sh ~/Downloads/
```

---

## Networking

| Command | What it does |
|---|---|
| `ifconfig` | Show network interfaces and IPs |
| `ping google.com` | Test connectivity |
| `curl https://example.com` | Fetch a URL |
| `wget https://example.com/file` | Download a file (install via Homebrew) |
| `netstat -an` | Show active connections |
| `lsof -i :8080` | Show what is using a specific port |
| `networksetup -listallnetworkservices` | List network interfaces |
| `ipconfig getifaddr en0` | Get IP of a specific interface |

```bash
# Find what is running on port 3000
lsof -i :3000

# Get your local IP
ipconfig getifaddr en0
```

---

## Homebrew

| Command | What it does |
|---|---|
| `brew install hugo` | Install a package |
| `brew uninstall hugo` | Remove a package |
| `brew update` | Update Homebrew |
| `brew upgrade` | Upgrade all packages |
| `brew upgrade hugo` | Upgrade a specific package |
| `brew list` | List installed packages |
| `brew search git` | Search for a package |
| `brew doctor` | Check for issues |
| `brew cleanup` | Remove old versions |
| `brew install --cask app` | Install a GUI application |

---

## Process Management

| Command | What it does |
|---|---|
| `ps aux` | List all running processes |
| `ps aux \| grep nginx` | Find a specific process |
| `kill PID` | Stop a process by ID |
| `kill -9 PID` | Force kill a process |
| `killall Finder` | Kill process by name |

```bash
# Restart Finder if it freezes
killall Finder
```

---

## zsh Tips

macOS uses zsh by default. A few useful built-ins:

```bash
# View command history
history

# Search history
history | grep docker

# Run previous command with sudo
sudo !!

# Clear terminal
clear   # or CTRL+L

# Expand path with Tab
cd ~/Doc[TAB]   # autocompletes to ~/Documents/
```

---

## Useful Keyboard Shortcuts

| Shortcut | What it does |
|---|---|
| `CTRL+C` | Cancel current command |
| `CTRL+L` | Clear terminal |
| `CTRL+A` | Jump to beginning of line |
| `CTRL+E` | Jump to end of line |
| `CTRL+U` | Clear current line |
| `↑ / ↓` | Navigate command history |
| `TAB` | Autocomplete |
| `CMD+T` | New terminal tab |
| `CMD+K` | Clear terminal (macOS Terminal.app) |

---

## Related Links

- [Install Homebrew on macOS](/guides/operating-systems/macos/install-homebrew-macos/) — get the macOS package manager first
- [Bash/Zsh Aliases on macOS](/guides/operating-systems/macos/aliases-macos/) — speed up your workflow with custom shortcuts
- [Apple Terminal User Guide](https://support.apple.com/guide/terminal/welcome/mac) — official Apple docs
