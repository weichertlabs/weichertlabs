---
title: "Part 12 – Uptime Kuma and Syncthing"
description: Deploying Uptime Kuma for service monitoring and Syncthing for file sync between devices — the final pieces of the WCP stack.
date: 2026-03-31
weight: 12
tags: [ubuntu, uptime-kuma, syncthing, docker, monitoring, file-sync, self-hosted]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

This final part of the WCP core stack adds two services: Uptime Kuma for monitoring all your services and alerting when something goes down, and Syncthing for continuous file sync between your devices without relying on any cloud.

---

## Prerequisites

- ✅ Docker and the `wcp-network` network in place
- ✅ Caddy configured with `status.wcp`
- ✅ Tailscale connected

→ Follow [Part 2](../part-02-docker-tailscale/) and [Part 3](../part-03-caddy/) first.

---

## Part A – Uptime Kuma

Uptime Kuma monitors all your WCP services and sends alerts when something goes down — via Telegram, email, Slack, ntfy, or dozens of other notification channels.

### Step 1 – Create the folder

```bash
mkdir -p /opt/docker/uptime-kuma
cd /opt/docker/uptime-kuma
```

### Step 2 – Create the compose.yml

```bash
nano compose.yml
```

```yaml
services:
  uptime-kuma:
    image: louislam/uptime-kuma:latest
    container_name: uptime-kuma
    volumes:
      - ./data:/app/data
    networks:
      - wcp-network
    restart: unless-stopped

networks:
  wcp-network:
    external: true
```

### Step 3 – Start Uptime Kuma

```bash
docker compose up -d
```

### Step 4 – Access the dashboard

```
http://status.wcp
```

Create an admin account on the first visit.

### Step 5 – Add monitors

Click **Add New Monitor** for each service:

| Service | Type | URL |
|---|---|---|
| Nextcloud | HTTP | `http://nextcloud:80` |
| Jellyfin | HTTP | `http://jellyfin:8096` |
| Immich | HTTP | `http://immich-server:2283` |
| Ollama | HTTP | `http://ollama:11434` |
| Vaultwarden | HTTP | `http://vaultwarden:80` |
| Caddy | HTTP | `http://caddy:80` |

Since everything is on the same `wcp-network`, Uptime Kuma can reach all services by container name.

### Step 6 – Set up notifications

Go to **Settings → Notifications → Add Notification**

**Telegram (recommended):**
1. Create a Telegram bot via [@BotFather](https://t.me/BotFather) — note the API token
2. Start a chat with your bot and send a message
3. Get your chat ID: `https://api.telegram.org/bot<TOKEN>/getUpdates`
4. In Uptime Kuma, add Telegram notification with your token and chat ID

Now you'll get a Telegram message whenever a service goes down or recovers.

---

## Part B – Syncthing

Syncthing keeps files in sync between your devices without any central server — everything goes peer-to-peer, encrypted, directly between your machines.

**Use cases in WCP:**
- Sync documents between Mac Studio and the WCP machine
- Keep project files in sync across all devices
- Sync configuration files between machines
- Backup important folders from your Mac to the WCP machine

### Step 1 – Create the folder

```bash
mkdir -p /opt/docker/syncthing/config
mkdir -p /opt/docker/syncthing/data
cd /opt/docker/syncthing
```

### Step 2 – Create the compose.yml

```bash
nano compose.yml
```

```yaml
services:
  syncthing:
    image: syncthing/syncthing:latest
    container_name: syncthing
    environment:
      - PUID=1000
      - PGID=1000
    volumes:
      - ./config:/var/syncthing/config
      - ./data:/var/syncthing/data
      - /opt:/sync/opt                  # sync service configs
      - /home/patrik/Documents:/sync/documents
    ports:
      - "22000:22000/tcp"
      - "22000:22000/udp"
      - "21027:21027/udp"
    networks:
      - wcp-network
    restart: unless-stopped

networks:
  wcp-network:
    external: true
```

**Note:** Syncthing needs its sync ports exposed directly — it doesn't work well behind a reverse proxy. Access the web UI via direct port instead of Caddy.

### Step 3 – Add Caddy entry for the web UI

Add to your Caddyfile:

```caddyfile
sync.wcp {
    reverse_proxy syncthing:8384
}
```

Reload Caddy:

```bash
docker exec caddy caddy reload --config /etc/caddy/Caddyfile
```

### Step 4 – Start Syncthing

```bash
docker compose up -d
```

### Step 5 – Access the web UI

```
http://sync.wcp
```

### Step 6 – Install Syncthing on your other devices

**macOS:**

```bash
brew install syncthing
brew services start syncthing
```

Access the Mac Syncthing UI at `http://localhost:8384`

**iOS/Android:** Search for **Möbius Sync** (iOS) or **Syncthing** (Android) in the app store.

### Step 7 – Connect devices

**On the WCP machine:**
1. Go to `http://sync.wcp`
2. Note the **Device ID** shown in the top right

**On your Mac:**
1. Go to `http://localhost:8384`
2. Click **Add Remote Device**
3. Enter the WCP machine's Device ID
4. Click **Save**

Accept the connection request on the WCP machine when it appears.

### Step 8 – Share folders

**On the Mac Syncthing:**
1. Click **Add Folder**
2. Select the folder you want to sync (e.g. `~/Documents/Projects`)
3. Under **Sharing**, enable sharing with the WCP machine
4. Click **Save**

Accept the folder on the WCP machine and set the local path to `/sync/documents` (or wherever you want it stored).

Files now sync automatically whenever both devices are online — via Tailscale if remote, directly if on the same network.

---

## WCP Stack complete!

With Part 12 done, the full WCP core stack is running:

| ✅ | Service | URL |
|---|---|---|
| ✅ | Nextcloud | `http://nextcloud.wcp` |
| ✅ | Immich | `http://immich.wcp` |
| ✅ | Jellyfin | `http://jellyfin.wcp` |
| ✅ | Ollama + Open WebUI | `http://ai.wcp` |
| ✅ | ComfyUI | `http://100.x.x.x:8188` |
| ✅ | Sunshine | `https://100.x.x.x:47990` |
| ✅ | Vaultwarden | `https://vault.wcp` |
| ✅ | Uptime Kuma | `http://status.wcp` |
| ✅ | Syncthing | `http://sync.wcp` |

All accessible via Tailscale from anywhere. No subscriptions. No data leaving your home. 🎉

---

## What's next

The core stack is complete. Future parts will cover:

- GPU sharing and performance tuning
- Automated backups and maintenance scripts
- Track B — Claude Code builds the same stack via SSH

---

## Related guides

- [Part 3 – Caddy Reverse Proxy](../part-03-caddy/) — reverse proxy for all services
- [Tailscale – Getting Started](/guides/networking/tailscale-getting-started/) — access everything remotely
- [Uptime Kuma GitHub](https://github.com/louislam/uptime-kuma) — official docs
- [Syncthing Documentation](https://docs.syncthing.net) — official docs
