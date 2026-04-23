---
title: Apple Containers on macOS – Early Setup and First Container
description: Apple's open-source container CLI runs Linux containers as lightweight virtual machines on Apple Silicon. This guide covers installation and first steps.
date: 2025-01-01
tags: [docker, containers, macos, apple-silicon]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

{{< callout type="info" >}}
**Early-stage project.** Apple Container was announced at WWDC25 and is still in active development. Full support targets macOS 26 Tahoe. It runs on macOS 15 Sequoia with some limitations. Expect the CLI and features to evolve.
{{< /callout >}}

Apple's `container` is an open-source CLI tool written in Swift that runs Linux containers as lightweight virtual machines on macOS. Announced at WWDC25, it is Apple's native answer to Docker Desktop — no third-party runtime required.

The key difference from Docker and OrbStack: each container gets its own isolated micro-VM instead of sharing a single Linux VM. This gives stronger security isolation but means some Docker features (like container-to-container networking on macOS 15) are not yet available.

It uses standard OCI images — you pull from Docker Hub and any other container registry just like with Docker.

## Requirements

- Apple Silicon Mac (M1 or later) — Intel is not supported
- macOS 15 Sequoia or later (macOS 26 Tahoe recommended for full functionality)
- Admin access for installation

---

## Step 1 – Download and install

Go to the [apple/container releases page](https://github.com/apple/container/releases) and download the latest signed `.pkg` installer.

Double-click the `.pkg` file and follow the prompts. Enter your admin password when asked. This installs the `container` binary to `/usr/local/bin/container`.

Verify the install:

```bash
container --version
```

---

## Step 2 – Pull an image

On first use, the tool will download a Linux kernel. Confirm when prompted.

Pull Alpine Linux:

```bash
container image pull alpine:latest
```

List downloaded images:

```bash
container image ls
```

---

## Step 3 – Run your first container

```bash
container run alpine:latest echo "Hello from Apple Containers"
```

You should see the output printed, then the container exits.

---

## Step 4 – Run an interactive container

```bash
container run -it alpine:latest sh
```

Inside the container:

```bash
uname -a
apk add curl
exit
```

Note: `uname -a` shows you are inside a Linux environment — fully isolated from the host.

---

## Step 5 – Run a container in the background

```bash
container run -d -p 8080:80 --name my-nginx nginx:latest
```

Open [http://localhost:8080](http://localhost:8080) to verify it is running.

---

## Useful commands

| Command | What it does |
|---|---|
| `container run -it image sh` | Run an interactive container |
| `container run -d -p 8080:80 image` | Run in the background with port mapping |
| `container ls` | List running containers |
| `container ls -a` | List all containers including stopped |
| `container stop name` | Stop a container |
| `container rm name` | Remove a container |
| `container image ls` | List downloaded images |
| `container image pull image:tag` | Pull an image |
| `container image rm image` | Remove an image |

> **Note:** Unlike Docker, the command to list containers is `container ls`, not `container ps`.

---

## How it differs from Docker

| | Docker / OrbStack | Apple Container |
|---|---|---|
| Container isolation | Shared Linux VM | One micro-VM per container |
| Host OS | macOS, Linux, Windows | macOS only |
| CPU support | Apple Silicon + Intel | Apple Silicon only |
| Docker Compose | Yes | Not yet |
| Container networking | Full | Limited on macOS 15 |
| Maturity | Production ready | Early-stage |

For daily use and Docker Compose workflows, OrbStack is still the better option. Apple Container is worth exploring if you want stronger isolation or are curious about where macOS containerization is heading.

---

## Related Links

- [apple/container on GitHub](https://github.com/apple/container)
- [WWDC25 – Meet Containerization](https://developer.apple.com/videos/play/wwdc2025/346/)
- [OrbStack on macOS – Install and First Container](/guides/containers/docker/orbstack-guide/)
- [Install Docker and Docker Compose on Linux](/guides/containers/docker/docker-compose-linux/)
