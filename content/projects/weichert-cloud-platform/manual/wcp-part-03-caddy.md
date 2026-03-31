---
title: "Part 3 – Caddy Reverse Proxy"
description: Setting up Caddy as a reverse proxy on the WCP machine — clean URLs and automatic HTTPS for every service, without touching a certificate manually.
date: 2026-03-31
tags: [ubuntu, caddy, reverse-proxy, https, docker, self-hosted]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

Without a reverse proxy, every service on the WCP machine runs on its own port — Nextcloud on 8080, Jellyfin on 8096, Vaultwarden on 8200, and so on. Caddy solves this by sitting in front of all services and routing traffic based on the domain name, with automatic HTTPS.

After this part, every service gets a clean URL like `nextcloud.wcp` instead of `192.168.1.50:8080`.

---

## Prerequisites

- ✅ Docker and Docker Compose installed
- ✅ Tailscale connected
- ✅ The `wcp-network` Docker network created

→ Follow [Part 2 – Docker and Tailscale](/projects/weichert-cloud-platform/manual/part-02-docker-tailscale/) first.

---

## How Caddy works in this setup

Caddy runs as a Docker container on the same `wcp-network` as all other services. When a request comes in for `nextcloud.wcp`, Caddy routes it to the Nextcloud container — the browser never needs to know which port Nextcloud is on.

For HTTPS, two options:

**Option A – Local only (Tailscale + internal DNS)**
Use Tailscale's MagicDNS and internal hostnames. No public domain needed. HTTPS via Caddy's self-signed certificates or a local CA.

**Option B – Public domain**
If you have a domain name, Caddy automatically provisions Let's Encrypt certificates. Zero configuration needed.

This guide covers Option A — local-only access via Tailscale. Option B is straightforward to add later if needed.

---

## Step 1 – Create the Caddy folder

```bash
mkdir -p /opt/docker/caddy/data
mkdir -p /opt/docker/caddy/config
cd /opt/docker/caddy
```

---

## Step 2 – Create the Caddyfile

```bash
nano Caddyfile
```

Add the following — this will grow as services are added throughout the series:

```caddyfile
# Global options
{
    admin off
    auto_https off
}

# Nextcloud
nextcloud.wcp {
    reverse_proxy nextcloud:80
}

# Jellyfin
jellyfin.wcp {
    reverse_proxy jellyfin:8096
}

# Immich
immich.wcp {
    reverse_proxy immich-server:2283
}

# Ollama Web UI
ai.wcp {
    reverse_proxy open-webui:8080
}

# Vaultwarden
vault.wcp {
    reverse_proxy vaultwarden:80
}

# Uptime Kuma
status.wcp {
    reverse_proxy uptime-kuma:3001
}
```

{{< callout type="info" >}}
`auto_https off` disables automatic HTTPS for now since we're using Tailscale for secure access. Each service is reachable over the Tailscale network without exposing ports to the internet.
{{< /callout >}}

---

## Step 3 – Create the compose.yml

```bash
nano compose.yml
```

```yaml
services:
  caddy:
    image: caddy:latest
    container_name: caddy
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - ./data:/data
      - ./config:/config
    networks:
      - wcp-network
    restart: unless-stopped

networks:
  wcp-network:
    external: true
```

---

## Step 4 – Start Caddy

```bash
docker compose up -d
docker compose logs -f
```

You should see Caddy start without errors. Press `CTRL+C` to exit logs.

---

## Step 5 – Set Up Local DNS

For the hostnames like `nextcloud.wcp` to resolve, you need local DNS. Two options:

### Option A – Edit /etc/hosts on each client machine

On your Mac, add entries to `/etc/hosts`:

```bash
sudo nano /etc/hosts
```

Add (replace with your WCP machine's Tailscale IP):

```
100.x.x.x   nextcloud.wcp
100.x.x.x   jellyfin.wcp
100.x.x.x   immich.wcp
100.x.x.x   ai.wcp
100.x.x.x   vault.wcp
100.x.x.x   status.wcp
```

### Option B – Use Tailscale DNS (Recommended)

In the Tailscale admin console under **DNS → Nameservers**, add a custom nameserver pointing to your WCP machine. This automatically resolves `.wcp` hostnames for all Tailscale-connected devices.

---

## Adding new services to Caddy

Every time a new service is added in this series, add a block to the Caddyfile:

```caddyfile
newservice.wcp {
    reverse_proxy container-name:port
}
```

Then reload Caddy without restarting:

```bash
docker exec caddy caddy reload --config /etc/caddy/Caddyfile
```

---

## Useful commands

| Command | What it does |
|---|---|
| `docker compose up -d` | Start Caddy |
| `docker compose logs -f` | Follow live logs |
| `docker exec caddy caddy reload --config /etc/caddy/Caddyfile` | Reload config without restart |
| `docker exec caddy caddy validate --config /etc/caddy/Caddyfile` | Validate config for errors |

---

## What's next

With Caddy in place, every service that gets added automatically gets a clean URL. Part 4 deploys Nextcloud — your private Google Drive.

**Up next:** Part 4 – Nextcloud *(coming soon)*

---

## Related guides

- [Tailscale – Getting Started](/guides/networking/tailscale-getting-started/) — secure access to all services
- [Install Docker and Docker Compose on Linux](/guides/containers/docker/docker-compose-linux/) — Docker basics
