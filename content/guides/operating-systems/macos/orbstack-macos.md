---
title: OrbStack – Docker on macOS Done Right
description: How to install and use OrbStack on macOS — a fast, lightweight alternative to Docker Desktop with built-in Docker and Linux machine support.
date: 2026-03-27
tags: [macos, orbstack, docker, containers, linux]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

OrbStack is a fast, lightweight way to run Docker containers and Linux machines on macOS. It is a drop-in replacement for Docker Desktop — same Docker CLI, same `compose.yml` files — but uses significantly less RAM, starts in seconds, and feels native on Apple Silicon.

**Why OrbStack instead of Docker Desktop:**
- Starts in 2 seconds instead of 30+
- Uses far less RAM and CPU in the background
- Native Apple Silicon support
- Built-in Linux machine support (run a full Linux VM alongside containers)
- Clean, simple menu bar app
- Free for personal use

## Requirements

- macOS 13 Ventura or later
- Apple Silicon (M1 or later) or Intel Mac
- [Homebrew](/guides/operating-systems/macos/install-homebrew-macos/) installed (recommended)

---

## Step 1 – Install OrbStack

**Via Homebrew (recommended):**

```bash
brew install --cask orbstack
```

**Or download directly from:**
[orbstack.dev](https://orbstack.dev)

---

## Step 2 – First Launch

Open OrbStack from your Applications folder or Spotlight. On first launch it will:

1. Ask for permission to install its helper
2. Set up the Docker socket automatically
3. Appear as a menu bar icon

That's it — Docker is now available in your terminal.

---

## Step 3 – Verify Docker Works

```bash
docker --version
docker compose version
docker run hello-world
```

Everything works exactly the same as Docker Desktop. Your existing `compose.yml` files, images, and workflows need no changes.

---

## Running Linux Machines

OrbStack includes a built-in Linux machine feature — lightweight VMs that start in seconds:

```bash
# Create and start an Ubuntu machine
orb create ubuntu my-ubuntu

# SSH into it
orb shell my-ubuntu

# List all machines
orb list
```

This is useful for testing Linux configurations without spinning up a full Proxmox VM.

---

## OrbStack CLI

OrbStack comes with the `orb` CLI tool:

| Command | What it does |
|---|---|
| `orb list` | List all Linux machines |
| `orb create ubuntu name` | Create a new Ubuntu machine |
| `orb shell name` | SSH into a machine |
| `orb start name` | Start a machine |
| `orb stop name` | Stop a machine |
| `orb delete name` | Delete a machine |
| `orb info` | Show OrbStack system info |

---

## Docker Compose Works as Normal

Your existing Docker Compose workflows work without any changes:

```bash
cd ~/docker/nginx
docker compose up -d
docker compose logs -f
docker compose down
```

OrbStack handles everything in the background — no configuration needed.

---

## Menu Bar Controls

The OrbStack menu bar icon gives you quick access to:
- Start/stop the OrbStack engine
- See running containers and machines
- Open the OrbStack dashboard (web UI for containers)
- View resource usage

---

## Migrating from Docker Desktop

If you were previously using Docker Desktop:

1. Quit Docker Desktop
2. Install OrbStack
3. Open OrbStack — it automatically imports your existing containers and images
4. Uninstall Docker Desktop if you no longer need it

```bash
# Uninstall Docker Desktop via Homebrew if installed that way
brew uninstall --cask docker
```

---

## Pricing

OrbStack is **free for personal use**. A paid license is required for commercial use. For home labs and personal projects — completely free.

---

## Related Links

- [OrbStack Official Site](https://orbstack.dev) — documentation and downloads
- [Install Docker and Docker Compose on Linux](/guides/containers/docker/docker-compose-linux/) — Docker on Linux servers
- [Essential macOS Terminal Commands](/guides/operating-systems/macos/essential-macos-terminal-commands/) — macOS terminal basics
