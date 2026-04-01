---
title: "Part 3 – Caddy Reverse Proxy"
description: Setting up Caddy as a reverse proxy on the WCP machine — clean URLs and HTTPS for every service using Tailscale certificates.
date: 2026-03-31
weight: 3
tags: [ubuntu, caddy, reverse-proxy, https, docker, tailscale, self-hosted]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

Without a reverse proxy, every service runs on its own port — Nextcloud on 8080, Jellyfin on 8096, Vaultwarden on 8200, and so on. Caddy solves this by routing traffic based on the domain name, with HTTPS handled automatically via Tailscale certificates.

After this part, every service gets a clean URL like `nextcloud.wcp` with valid HTTPS — no certificate warnings, no browser complaints.

---

## Prerequisites

- ✅ Docker and the `wcp-network` Docker network created
- ✅ Tailscale connected and running
- ✅ MagicDNS enabled in Tailscale admin console

→ Follow [Part 2](../part-02-docker-tailscale/) first.

---

## How this setup works

Caddy runs as a Docker container on the `wcp-network`. Services that require HTTPS (Nextcloud, Vaultwarden) get Tailscale-issued certificates — trusted by all your Tailscale-connected devices without any warnings. Other services run on HTTP since they don't need secure context.

---

## Step 1 – Get Tailscale certificates

Tailscale can issue valid HTTPS certificates for your machine's Tailscale hostname:

```bash
sudo tailscale cert wcp
```

This creates two files:
```
/var/lib/tailscale/certs/wcp.crt
/var/lib/tailscale/certs/wcp.key
```

{{< callout type="info" >}}
Replace `wcp` with your actual Tailscale machine name if you named it differently. Check with `tailscale status`.
{{< /callout >}}

---

## Step 2 – Create the Caddy folder

```bash
mkdir -p /opt/docker/caddy/data
mkdir -p /opt/docker/caddy/config
cd /opt/docker/caddy
```

---

## Step 3 – Create the Caddyfile

```bash
nano Caddyfile
```

```caddyfile
# Global options
{
    admin off
    auto_https off
}

# Services requiring HTTPS (secure context)
https://nextcloud.wcp {
    tls /certs/wcp.crt /certs/wcp.key
    reverse_proxy nextcloud:80
}

https://vault.wcp {
    tls /certs/wcp.crt /certs/wcp.key
    reverse_proxy vaultwarden:80
}

# Services that work fine on HTTP
jellyfin.wcp {
    reverse_proxy jellyfin:8096
}

immich.wcp {
    reverse_proxy immich-server:2283
}

ai.wcp {
    reverse_proxy open-webui:8080
}

status.wcp {
    reverse_proxy uptime-kuma:3001
}

sync.wcp {
    reverse_proxy syncthing:8384
}
```

---

## Step 4 – Create the compose.yml

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
      - /var/lib/tailscale/certs:/certs:ro
    networks:
      - wcp-network
    restart: unless-stopped

networks:
  wcp-network:
    external: true
```

**Key addition:** `/var/lib/tailscale/certs:/certs:ro` mounts the Tailscale certificates into the Caddy container read-only.

---

## Step 5 – Start Caddy

```bash
docker compose up -d
docker compose logs -f
```

Press `CTRL+C` when running.

---

## Step 6 – Set Up Local DNS

For hostnames like `nextcloud.wcp` to resolve on your devices, add entries to the hosts file on each machine.

**On your Mac:**

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
100.x.x.x   sync.wcp
```

**On iPhone/iPad via Tailscale DNS:**

In the Tailscale admin console under **DNS → Nameservers**, add a custom nameserver pointing to your WCP machine. This resolves `.wcp` hostnames automatically for all Tailscale-connected devices including phones.

---

## Renewing Tailscale certificates

Tailscale certificates expire and need to be renewed periodically. Automate this with a cron job:

```bash
sudo crontab -e
```

Add:

```
0 0 1 * * tailscale cert wcp && docker exec caddy caddy reload --config /etc/caddy/Caddyfile
```

This renews the cert and reloads Caddy on the 1st of every month.

---

## Adding new services

Every time a new service is added, add a block to the Caddyfile:

```caddyfile
# For services that don't need HTTPS
newservice.wcp {
    reverse_proxy container-name:port
}

# For services that need HTTPS
https://newservice.wcp {
    tls /certs/wcp.crt /certs/wcp.key
    reverse_proxy container-name:port
}
```

Then reload Caddy:

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
| `docker exec caddy caddy validate --config /etc/caddy/Caddyfile` | Check config for errors |
| `sudo tailscale cert wcp` | Renew Tailscale certificate |

---

## What's next

With Caddy in place and HTTPS working, Part 4 deploys Nextcloud — your private Google Drive.

**Up next:** [Part 4 – Nextcloud](../part-04-nextcloud/) *(coming soon)*

---

## Related guides

- [Part 2 – Docker and Tailscale](../part-02-docker-tailscale/) — Tailscale setup
- [Tailscale – Getting Started](/guides/networking/tailscale-getting-started/) — Tailscale basics
- [Caddy Documentation](https://caddyserver.com/docs/) — official docs
