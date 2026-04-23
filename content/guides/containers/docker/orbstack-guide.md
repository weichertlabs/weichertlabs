---
title: OrbStack on macOS – Install and First Container
description: Install OrbStack on macOS and run your first Docker container. A fast, lightweight alternative to Docker Desktop with native Apple Silicon support.
date: 2025-01-01
tags: [docker, orbstack, macos, containers]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

OrbStack is a fast, lightweight Docker runtime for macOS. It replaces Docker Desktop — uses the same Docker CLI and `compose.yml` files — but starts in seconds and uses far less RAM. If you're running containers on a Mac, this is the setup to use.

## Requirements

- macOS 13 Ventura or later
- Apple Silicon or Intel Mac
- [Homebrew](https://weichertlabs.com/guides/operating-systems/macos/install-homebrew-macos/) installed

---

## Step 1 – Install OrbStack

```bash
brew install --cask orbstack
```

Or download directly from [orbstack.dev](https://orbstack.dev).

Open OrbStack from Applications or Spotlight. On first launch it installs its helper and sets up the Docker socket automatically. A menu bar icon appears when it's ready.

---

## Step 2 – Verify Docker is working

```bash
docker --version
docker compose version
```

Run a quick test:

```bash
docker run hello-world
```

If you see the Hello from Docker message — everything is working.

---

## Step 3 – Run your first real container

Start an Nginx web server:

```bash
docker run -d -p 8080:80 --name my-nginx nginx
```

Open [http://localhost:8080](http://localhost:8080) — you should see the Nginx welcome page.

Stop and remove it:

```bash
docker stop my-nginx
docker rm my-nginx
```

---

## Step 4 – Run an interactive container

```bash
docker run -it ubuntu bash
```

Inside the container:

```bash
apt update && apt install -y curl
curl --version
exit
```

---

## Step 5 – Docker Compose

OrbStack is fully compatible with Docker Compose. Your existing `compose.yml` files work without any changes:

```bash
cd ~/docker/your-project
docker compose up -d
docker compose logs -f
docker compose down
```

---

## Useful commands

| Command | What it does |
|---|---|
| `docker ps` | List running containers |
| `docker ps -a` | List all containers including stopped |
| `docker images` | List downloaded images |
| `docker stop name` | Stop a container |
| `docker rm name` | Remove a container |
| `docker rmi image` | Remove an image |
| `docker logs name` | View container logs |

---

## Notes

- OrbStack automatically replaces Docker Desktop — no extra configuration needed
- Existing images and containers are imported automatically if you had Docker Desktop before
- Free for personal use — a paid license is required for commercial use
- The OrbStack menu bar icon gives quick access to running containers, resource usage, and the built-in dashboard

---

## Related Links

- [OrbStack Official Site](https://orbstack.dev)
- [OrbStack on macOS – Full Overview](/guides/operating-systems/macos/orbstack-macos/)
- [Install Docker and Docker Compose on Linux](/guides/containers/docker/docker-compose-linux/)
- [Portainer – Web UI for Docker](/guides/containers/docker/portainer-docker/)
