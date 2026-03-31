---
title: "Part 5 – Immich"
description: Deploying Immich on the WCP machine — a self-hosted Google Photos alternative with AI-powered face recognition, object detection, and automatic backup from your phone.
date: 2026-03-31
tags: [ubuntu, immich, docker, self-hosted, photos, ai]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

Immich is a self-hosted photo and video backup solution with a Google Photos-like interface. It automatically backs up photos from your phone over Tailscale, organises them by date and location, and uses machine learning for face recognition and object search — all running locally on your own hardware.

**What you get:**
- Automatic photo backup from iPhone, Android, and iPad
- Face recognition and object search
- Timeline view, albums, and sharing
- Video support
- iOS and Android apps

---

## Prerequisites

- ✅ Docker and the `wcp-network` network in place
- ✅ Caddy running with `immich.wcp` configured
- ✅ Plenty of storage space for photos

→ Follow [Part 2](/projects/weichert-cloud-platform/manual/part-02-docker-tailscale/) and [Part 3](/projects/weichert-cloud-platform/manual/part-03-caddy/) first.

---

## Step 1 – Create the Immich folder

```bash
mkdir -p /opt/docker/immich
cd /opt/docker/immich
```

---

## Step 2 – Download the official compose file

Immich provides an official `compose.yml` that is kept up to date. Use it directly:

```bash
wget https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml
wget https://github.com/immich-app/immich/releases/latest/download/example.env -O .env
```

---

## Step 3 – Configure the .env file

```bash
nano .env
```

Key settings to update:

```env
# Database password — change this
DB_PASSWORD=change-this-password

# Where photos are stored on the host
UPLOAD_LOCATION=/opt/immich-library

# Your timezone
TZ=Europe/Stockholm
```

Leave the rest as defaults for now.

---

## Step 4 – Create the photo library directory

```bash
sudo mkdir -p /opt/immich-library
sudo chown -R $USER:$USER /opt/immich-library
```

---

## Step 5 – Add to the wcp-network

Open `docker-compose.yml` and add the external network at the bottom:

```bash
nano docker-compose.yml
```

Add to the `networks` section at the bottom of the file:

```yaml
networks:
  default:
    name: wcp-network
    external: true
```

---

## Step 6 – Start Immich

```bash
docker compose up -d
docker compose logs -f
```

The first start downloads machine learning models — this takes a few minutes. When logs stabilise, press `CTRL+C`.

---

## Step 7 – Access Immich

Open your browser:

```
http://immich.wcp
```

Create your admin account on the first visit.

---

## Step 8 – Install the mobile app

**iOS:** Search for **Immich** in the App Store.

**Android:** Available on Google Play or F-Droid.

In the app:
1. Set the server URL to `http://immich.wcp` (connected via Tailscale)
2. Log in with your account
3. Enable automatic backup in the app settings

Photos will now back up automatically whenever your phone is connected to Tailscale.

---

## Step 9 – GPU acceleration for machine learning (optional)

Immich's face recognition and object detection can use the GPU for faster processing. Add this to the `immich-machine-learning` service in `docker-compose.yml`:

```yaml
  immich-machine-learning:
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]
```

Restart to apply:

```bash
docker compose up -d
```

---

## Keeping Immich updated

Immich releases updates frequently. Update regularly:

```bash
cd /opt/docker/immich
docker compose pull
docker compose up -d
```

{{< callout type="info" >}}
Immich is under active development and releases updates often. Check the [Immich release notes](https://github.com/immich-app/immich/releases) before updating to be aware of any breaking changes.
{{< /callout >}}

---

## Related guides

- [Part 3 – Caddy Reverse Proxy](/projects/weichert-cloud-platform/manual/part-03-caddy/) — required for `immich.wcp` to work
- [Tailscale – Getting Started](/guides/networking/tailscale-getting-started/) — required for mobile app access
- [Immich Documentation](https://immich.app/docs/) — official docs
