---
title: Portainer – Web UI for Docker
description: How to install Portainer using Docker Compose and manage your containers, images, volumes, and networks through a clean web interface.
date: 2026-03-27
tags: [docker, portainer, containers, web-ui, linux]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

Portainer is a web-based management UI for Docker. Instead of managing containers from the terminal, you get a clean dashboard where you can start, stop, inspect, and deploy containers — all from your browser.

{{< callout type="info" >}}
New to Docker? Set it up first: [Install Docker and Docker Compose on Linux](/guides/containers/docker/docker-compose-linux/)
{{< /callout >}}

**What Portainer gives you:**
- Visual overview of all containers, images, volumes, and networks
- Deploy new containers and stacks from the UI
- View container logs and stats in real time
- Manage multiple Docker hosts from one place
- Free Community Edition covers everything you need for a home lab

---

## Step 1 – Create the Project Folder

```bash
mkdir -p ~/docker/portainer
cd ~/docker/portainer
```

---

## Step 2 – Create the compose.yml

```bash
nano compose.yml
```

Add the following:

```yaml
services:
  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    ports:
      - "9000:9000"
      - "9443:9443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data
    restart: unless-stopped

volumes:
  portainer_data:
```

**What the options mean:**
- `9000` — HTTP access (unencrypted)
- `9443` — HTTPS access (self-signed certificate)
- `/var/run/docker.sock` — gives Portainer access to the Docker engine
- `portainer_data` — named volume that stores Portainer's configuration

---

## Step 3 – Start Portainer

```bash
docker compose up -d
```

Check that it started:

```bash
docker compose logs -f
```

Press `CTRL+C` to exit the log view.

---

## Step 4 – Access the Web Interface

Open your browser and go to:

```
http://your-server-ip:9000
```

Or via HTTPS:

```
https://your-server-ip:9443
```

The first time you visit, Portainer will ask you to create an admin user. Set a strong password and click **Create user**.

On the next screen, select **Docker Standalone** and click **Connect** — Portainer will detect your local Docker environment automatically.

---

## What You Can Do in Portainer

**Containers** — start, stop, restart, remove, inspect logs and stats for any container

**Stacks** — deploy Docker Compose stacks directly from the UI — paste your `compose.yml` and click Deploy

**Images** — pull new images, remove old ones, inspect image layers

**Volumes** — view and manage Docker volumes

**Networks** — inspect and manage Docker networks

**Stats** — real-time CPU, memory, and network usage per container

---

## Deploy a Stack from the UI

Instead of using the terminal for `docker compose up`, you can deploy stacks directly in Portainer:

1. Go to **Stacks → Add stack**
2. Give it a name
3. Paste your `compose.yml` content
4. Click **Deploy the stack**

This is great for managing multiple services visually.

---

## Keep Portainer Updated

```bash
cd ~/docker/portainer
docker compose pull
docker compose up -d
```

---

## Useful Commands

| Command | What it does |
|---|---|
| `docker compose up -d` | Start Portainer |
| `docker compose down` | Stop Portainer |
| `docker compose pull` | Pull latest Portainer image |
| `docker compose logs -f` | Follow live logs |

---

## Tips

- **Bookmark the URL** — you'll visit Portainer often, make it easy to reach
- **Use HTTPS** on port 9443 for a more secure connection (self-signed cert is fine for home lab use)
- **Portainer Agent** — if you have multiple Docker hosts, install the Portainer Agent on each one and manage them all from a single Portainer instance
- **Combine with Tailscale** — access Portainer remotely without exposing port 9000 to the internet

---

## Related Links

- [Install Docker and Docker Compose on Linux](/guides/containers/docker/docker-compose-linux/) — install Docker first
- [Tailscale – Getting Started](/guides/networking/tailscale-getting-started/) — access Portainer remotely without port forwarding
- [Portainer Documentation](https://docs.portainer.io) — official docs
