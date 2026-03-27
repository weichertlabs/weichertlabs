---
title: Bash/Zsh Aliases on macOS
description: How to create and manage aliases in zsh on macOS — save time by turning long commands into short shortcuts you actually remember.
date: 2026-03-27
tags: [macos, terminal, aliases, zsh, productivity]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

An alias is a custom shortcut for a command you use often. Instead of typing `cd /Volumes/SN200/weichertlabs && git status` every time, you type `wl` and you're there.

This guide covers creating, managing, and organizing aliases in zsh on macOS.

---

## How Aliases Work

An alias maps a short name to a longer command:

```bash
alias gs="git status"
```

Now typing `gs` runs `git status`. Simple as that.

---

## Step 1 – Open Your zsh Config File

On macOS, zsh loads its configuration from `~/.zshrc`. Open it in your preferred editor:

```bash
nano ~/.zshrc
```

Or with VS Code:

```bash
code ~/.zshrc
```

---

## Step 2 – Add Your Aliases

Scroll to the bottom of the file and add your aliases. Each alias goes on its own line:

```bash
# Navigation shortcuts
alias ..="cd .."
alias ...="cd ../.."
alias ~="cd ~"
alias dl="cd ~/Downloads"
alias dt="cd ~/Desktop"
alias docs="cd ~/Documents"

# List files
alias ll="ls -la"
alias la="ls -la"
alias l="ls -lh"

# Git shortcuts
alias gs="git status"
alias ga="git add ."
alias gc="git commit -m"
alias gp="git push"
alias gpl="git pull"
alias gl="git log --oneline --graph"

# Quick edit config files
alias zshrc="nano ~/.zshrc"
alias reload="source ~/.zshrc"

# Networking
alias myip="curl ifconfig.me"
alias localip="ipconfig getifaddr en0"
alias flushdns="sudo dscacheutil -flushcache && sudo killall -HUP mDNSResponder"

# Homebrew
alias bu="brew update && brew upgrade"
alias bclean="brew cleanup"

# Docker
alias dps="docker ps"
alias dpsa="docker ps -a"
alias dcu="docker compose up -d"
alias dcd="docker compose down"
alias dcl="docker compose logs -f"

# System
alias cpu="top -o cpu"
alias mem="top -o rsize"
alias diskspace="df -h"
alias cleanup="find . -name '.DS_Store' -delete"
```

---

## Step 3 – Save and Reload

Save the file (`CTRL+O`, then `CTRL+X` in nano), then reload your config:

```bash
source ~/.zshrc
```

Your aliases are now active. Test one:

```bash
gs
```

---

## Aliases with Arguments

Regular aliases can't take arguments, but you can use a **function** instead:

```bash
# Create a directory and cd into it
mkcd() {
    mkdir -p "$1" && cd "$1"
}

# Search for text in files
search() {
    grep -r "$1" .
}

# Quick git commit with message
gcommit() {
    git add . && git commit -m "$1" && git push
}
```

Usage:

```bash
mkcd my-new-project    # creates folder and enters it
search "docker"        # searches for "docker" in current folder
gcommit "add new guide"  # adds, commits, and pushes in one command
```

---

## Organizing Your Aliases

As your alias list grows, it helps to split them into separate files:

```bash
# In ~/.zshrc, add this line:
source ~/.zsh_aliases
```

Then create `~/.zsh_aliases` and put all your aliases there. Keeps `~/.zshrc` clean.

---

## View All Active Aliases

```bash
alias
```

Search for a specific alias:

```bash
alias | grep git
```

---

## Remove an Alias (Temporarily)

```bash
unalias gs
```

This removes the alias for the current session only. It comes back on next terminal open.

To remove permanently, delete it from `~/.zshrc` and run `source ~/.zshrc`.

---

## Useful Starter Set

Here is a clean starter set you can copy straight into your `~/.zshrc`:

```bash
# ── Navigation ──────────────────────────────
alias ..="cd .."
alias ...="cd ../.."
alias ll="ls -la"
alias dl="cd ~/Downloads"

# ── Git ─────────────────────────────────────
alias gs="git status"
alias ga="git add ."
alias gp="git push"
alias gpl="git pull"
alias gl="git log --oneline --graph"

# ── Homebrew ────────────────────────────────
alias bu="brew update && brew upgrade"
alias bclean="brew cleanup"

# ── Docker ──────────────────────────────────
alias dps="docker ps"
alias dcu="docker compose up -d"
alias dcd="docker compose down"
alias dcl="docker compose logs -f"

# ── Network ─────────────────────────────────
alias myip="curl ifconfig.me"
alias localip="ipconfig getifaddr en0"

# ── Reload config ───────────────────────────
alias reload="source ~/.zshrc"
alias zshrc="nano ~/.zshrc"
```

---

## Related Links

- [Essential macOS Terminal Commands](/guides/operating-systems/macos/essential-macos-terminal-commands/) — learn the commands before aliasing them
- [Install Homebrew on macOS](/guides/operating-systems/macos/install-homebrew-macos/) — required for many tools referenced here
- [Bash/Zsh Aliases on Linux](/guides/operating-systems/linux/aliases-linux/) — same concept for Linux servers
