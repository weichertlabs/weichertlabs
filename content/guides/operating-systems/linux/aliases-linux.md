---
title: Bash Aliases on Linux
description: How to create and manage aliases in bash on Linux — turn long commands into short shortcuts that save time every single day.
date: 2026-03-27
tags: [linux, terminal, aliases, bash, productivity]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

An alias is a custom shortcut for a command you use often. Instead of typing `sudo systemctl restart nginx` every time, you just type `rnginx`. This guide covers creating, managing, and organizing aliases in bash on Linux (Ubuntu/Debian).

---

## How Aliases Work

An alias maps a short name to a longer command:

```bash
alias ll="ls -la"
```

Now typing `ll` runs `ls -la`. That's all there is to it.

---

## Step 1 – Open Your bash Config File

On Ubuntu and Debian, bash loads its configuration from `~/.bashrc`. Open it:

```bash
nano ~/.bashrc
```

---

## Step 2 – Add Your Aliases

Scroll to the bottom and add your aliases:

```bash
# Navigation
alias ..="cd .."
alias ...="cd ../.."
alias ll="ls -la"
alias la="ls -la"
alias l="ls -lh"

# Git
alias gs="git status"
alias ga="git add ."
alias gp="git push"
alias gpl="git pull"
alias gl="git log --oneline --graph"

# Docker
alias dps="docker ps"
alias dpsa="docker ps -a"
alias dcu="docker compose up -d"
alias dcd="docker compose down"
alias dcl="docker compose logs -f"
alias dprune="docker system prune -f"

# System
alias update="sudo apt update && sudo apt upgrade -y"
alias diskspace="df -h"
alias meminfo="free -h"
alias ports="ss -tulnp"
alias myip="curl ifconfig.me"

# Services
alias sstart="sudo systemctl start"
alias sstop="sudo systemctl stop"
alias srestart="sudo systemctl restart"
alias sstatus="sudo systemctl status"
alias senable="sudo systemctl enable"

# Quick edit
alias bashrc="nano ~/.bashrc"
alias reload="source ~/.bashrc"
```

---

## Step 3 – Save and Reload

Save with `CTRL+O`, exit with `CTRL+X`, then reload:

```bash
source ~/.bashrc
```

Test an alias:

```bash
ll
```

---

## Aliases with Arguments

Regular aliases can't accept arguments. Use a **function** instead:

```bash
# Create directory and cd into it
mkcd() {
    mkdir -p "$1" && cd "$1"
}

# Search for text recursively
search() {
    grep -r "$1" .
}

# Quick git commit, add and push
gcommit() {
    git add . && git commit -m "$1" && git push
}

# Show last N lines of a log file
log() {
    sudo tail -n "${2:-50}" /var/log/"$1"
}
```

Usage:

```bash
mkcd my-project         # creates and enters folder
search "error"          # searches current directory
gcommit "add guide"     # add, commit and push in one command
log syslog 100          # show last 100 lines of syslog
```

---

## Organizing Your Aliases

Keep `~/.bashrc` clean by splitting aliases into a separate file:

```bash
# Add this line to ~/.bashrc
if [ -f ~/.bash_aliases ]; then
    source ~/.bash_aliases
fi
```

Then create `~/.bash_aliases` and put all your aliases there.

---

## View All Active Aliases

```bash
alias
```

Search for a specific one:

```bash
alias | grep docker
```

---

## Remove an Alias (Temporarily)

```bash
unalias ll
```

Removes for current session only. Comes back on next login.

To remove permanently, delete it from `~/.bashrc` and run `source ~/.bashrc`.

---

## Useful Starter Set for Linux Servers

A clean set to get you started:

```bash
# ── Navigation ──────────────────────────────
alias ..="cd .."
alias ll="ls -la"

# ── System ──────────────────────────────────
alias update="sudo apt update && sudo apt upgrade -y"
alias diskspace="df -h"
alias meminfo="free -h"
alias ports="ss -tulnp"
alias myip="curl ifconfig.me"

# ── Services ────────────────────────────────
alias srestart="sudo systemctl restart"
alias sstatus="sudo systemctl status"

# ── Git ─────────────────────────────────────
alias gs="git status"
alias ga="git add ."
alias gp="git push"
alias gpl="git pull"

# ── Docker ──────────────────────────────────
alias dps="docker ps"
alias dcu="docker compose up -d"
alias dcd="docker compose down"
alias dcl="docker compose logs -f"

# ── Reload ──────────────────────────────────
alias reload="source ~/.bashrc"
alias bashrc="nano ~/.bashrc"
```

---

## Related Links

- [Essential Linux Commands](/guides/operating-systems/linux/essential-linux-commands/) — learn the commands before aliasing them
- [SSH Keys – The Right Way](/guides/operating-systems/linux/ssh-keys-linux/) — secure your server access
- [Bash/Zsh Aliases on macOS](/guides/operating-systems/macos/aliases-macos/) — same concept for macOS
