---
title: "Part 11 – Vaultwarden"
description: Deploying Vaultwarden on the WCP machine — a self-hosted Bitwarden-compatible password manager, accessible from all your devices via Tailscale.
date: 2026-03-31
tags: [ubuntu, vaultwarden, bitwarden, docker, password-manager, security, self-hosted]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

Vaultwarden is an unofficial, lightweight implementation of the Bitwarden server — fully compatible with all official Bitwarden apps. Run your own password manager on your own hardware, with all your passwords staying on your machine.

**Works with all Bitwarden clients:**
- iOS and Android apps
- Browser extensions (Chrome, Firefox, Safari)
- macOS desktop app
- Web vault

---

## Prerequisites

- ✅ Docker and the `wcp-network` network in place
- ✅ Caddy configured with `vault.wcp`
- ✅ HTTPS required — Bitwarden clients refuse to connect over plain HTTP

→ Follow [Part 2](../part-02-docker-tailscale/) and [Part 3](../part-03-caddy/) first.

{{< callout type="warning" >}}
Vaultwarden requires HTTPS to work. In this setup, Caddy handles HTTPS via Tailscale. If you're not using Caddy with HTTPS, set it up before proceeding.
{{< /callout >}}

---

## Step 1 – Create the folder

```bash
mkdir -p /opt/docker/vaultwarden/data
cd /opt/docker/vaultwarden
```

---

## Step 2 – Create the compose.yml

```bash
nano compose.yml
```

```yaml
services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    environment:
      - DOMAIN=https://vault.wcp
      - SIGNUPS_ALLOWED=true
      - ADMIN_TOKEN=${ADMIN_TOKEN}
      - TZ=Europe/Stockholm
    volumes:
      - ./data:/data
    networks:
      - wcp-network
    restart: unless-stopped

networks:
  wcp-network:
    external: true
```

---

## Step 3 – Create the .env file

Generate a secure admin token:

```bash
openssl rand -base64 48
```

Copy the output and add it to your `.env` file:

```bash
nano .env
```

```
ADMIN_TOKEN=your-generated-token-here
```

---

## Step 4 – Start Vaultwarden

```bash
docker compose up -d
docker compose logs -f
```

Press `CTRL+C` when the container is running.

---

## Step 5 – Access the web vault

Open your browser:

```
https://vault.wcp
```

### Create your account

1. Click **Create Account**
2. Enter your email and a strong master password
3. **Remember your master password** — it cannot be recovered if lost

### Disable new registrations

After creating your account, disable open registration to prevent others from signing up:

In `compose.yml`, change:

```yaml
- SIGNUPS_ALLOWED=false
```

Restart:

```bash
docker compose restart
```

---

## Step 6 – Access the admin panel

The admin panel lets you manage users, view statistics, and configure advanced settings:

```
https://vault.wcp/admin
```

Enter the `ADMIN_TOKEN` from your `.env` file to log in.

---

## Step 7 – Connect Bitwarden clients

**iOS / Android:**
1. Open the Bitwarden app
2. Tap the region selector (US flag or similar) at the top
3. Select **Self-hosted**
4. Enter server URL: `https://vault.wcp`
5. Log in with your email and master password

**Browser extension:**
1. Click the extension settings icon
2. Select **Self-hosted Environment**
3. Enter: `https://vault.wcp`
4. Log in

**macOS desktop app:**
Same as browser extension — find the self-hosted setting in the login screen.

---

## Step 8 – Enable TOTP (Two-Factor Authentication)

Vaultwarden supports TOTP (Google Authenticator, Authy, etc.) for two-factor authentication:

1. In the web vault, go to **Account Settings → Security → Two-step Login**
2. Enable **Authenticator App**
3. Scan the QR code with your authenticator app
4. Save the recovery code in a safe place

---

## Backup your vault

Your passwords are stored in `/opt/docker/vaultwarden/data/`. Back this up regularly:

```bash
# Manual backup
tar -czf vaultwarden-backup-$(date +%Y%m%d).tar.gz /opt/docker/vaultwarden/data/
```

Consider automating this with a cron job and storing backups on your PBS server.

---

## Keeping Vaultwarden updated

```bash
cd /opt/docker/vaultwarden
docker compose pull
docker compose up -d
```

---

## What's next

Part 12 wraps up the core WCP stack with Uptime Kuma for service monitoring and Syncthing for file sync between devices.

**Up next:** [Part 12 – Uptime Kuma and Syncthing](../part-12-uptime-kuma-syncthing/) *(coming soon)*

---

## Related guides

- [Part 3 – Caddy Reverse Proxy](../part-03-caddy/) — required for HTTPS on `vault.wcp`
- [Tailscale – Getting Started](/guides/networking/tailscale-getting-started/) — secure access from all devices
- [Vaultwarden Documentation](https://github.com/dani-garcia/vaultwarden/wiki) — official wiki
- [Bitwarden Apps](https://bitwarden.com/download/) — download official clients
