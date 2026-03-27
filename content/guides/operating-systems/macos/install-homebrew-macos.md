---
title: Install Homebrew on macOS
description: How to install Homebrew — the missing package manager for macOS — and use it to install and manage software from the terminal.
date: 2026-03-27
tags: [macos, homebrew, terminal, package-manager]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

Homebrew is the most popular package manager for macOS. It lets you install, update, and manage software directly from the terminal — no downloading installers, no drag-and-drop to Applications. Just one command and you're done.

If you plan to use your Mac for development, infrastructure work, or any kind of technical tinkering, Homebrew is essentially required. Most tools in future guides on this site assume you have it installed.

## Requirements

- macOS 13 Ventura or later (works on Apple Silicon and Intel)
- Terminal access (Terminal.app or iTerm2)
- An internet connection

---

## Step 1 – Install Homebrew

Open Terminal and run the official install script:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

The script will ask for your Mac password and may install Apple's Xcode Command Line Tools if not already present. Follow the prompts and wait for it to finish.

---

## Step 2 – Add Homebrew to Your PATH (Apple Silicon only)

If you're on an Apple Silicon Mac (M1, M2, M3, M4 or later), Homebrew installs to `/opt/homebrew` instead of `/usr/local`. Add it to your shell profile:

```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zshrc
source ~/.zshrc
```

On Intel Macs this is not needed — Homebrew is already in your PATH automatically.

---

## Step 3 – Verify the Installation

```bash
brew --version
```

You should see something like:

```
Homebrew 4.x.x
```

Run a health check to make sure everything is set up correctly:

```bash
brew doctor
```

If it says `Your system is ready to brew` — you're all set.

---

## Basic Usage

Install a package:
```bash
brew install hugo
```

Update Homebrew and all packages:
```bash
brew update && brew upgrade
```

Search for a package:
```bash
brew search git
```

Remove a package:
```bash
brew uninstall hugo
```

List installed packages:
```bash
brew list
```

---

## Formulae vs Casks

Homebrew has two types of packages:

- **Formulae** – command-line tools and libraries (e.g. `hugo`, `git`, `wget`)
- **Casks** – macOS applications with a GUI (e.g. `brew install --cask visual-studio-code`)

```bash
# Install a CLI tool
brew install wget

# Install a GUI app
brew install --cask visual-studio-code
```

---

## Related Links

- [Homebrew Official Site](https://brew.sh) — documentation and package search
- [Homebrew Formulae](https://formulae.brew.sh) — browse all available packages
