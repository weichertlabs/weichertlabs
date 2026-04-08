---
title: OrbStack on macOS – Install and First Container
description: Install OrbStack using Homebrew and run your first container from the terminal.
date: 2026-04-08
tags: [orbstack, docker, macos, containers]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and commands are provided for educational purposes only. Always review commands before running them.
{{< /callout >}}

OrbStack is a fast and lightweight alternative to Docker Desktop on macOS. It uses less resources, starts instantly, and integrates better with the system.

This guide shows how to install OrbStack and run your first container.

---

## Why OrbStack?

- Faster startup than Docker Desktop  
- Lower CPU and RAM usage  
- Native macOS integration  
- Built-in terminal and container management  
- No heavy virtualization overhead  

---

## Requirements

- macOS (Apple Silicon or Intel)  
- Homebrew installed  

---

## Step 1 – Install OrbStack

Install using Homebrew:

```bash
brew install --cask orbstack
```

Start OrbStack:

```bash
open -a OrbStack
```

Wait until the service is running (menu bar icon will appear).

---

## Step 2 – Verify Docker is working

OrbStack provides a Docker-compatible CLI.

Check version:

```bash
docker version
```

If it returns version info → everything is working.

---

## Step 3 – Run your first container

Run a simple test container:

```bash
docker run hello-world
```

You should see a confirmation message from Docker.

---

## Step 4 – Run an interactive container

Example using Ubuntu:

```bash
docker run -it ubuntu bash
```

Inside the container:

```bash
apt update
apt install -y curl
```

Exit container:

```bash
exit
```

---

## Step 5 – List containers

```bash
docker ps
```

Show all containers:

```bash
docker ps -a
```

---

## Notes

- OrbStack automatically replaces Docker Desktop  
- No need to manually configure a VM  
- Works with existing Docker commands and tools  

---

## Related Links

- https://orbstack.dev/
- https://docs.docker.com/
