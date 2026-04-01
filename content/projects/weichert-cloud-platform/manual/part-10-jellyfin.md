---
title: "Part 10 – Jellyfin"
description: Deploying Jellyfin on the WCP machine — your own Netflix, streaming your media library to any device on your network or via Tailscale.
date: 2026-03-31
weight: 10
tags: [ubuntu, jellyfin, docker, media, streaming, self-hosted, nvidia]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

Jellyfin is a free, open-source media server — your own Netflix running on your own hardware. It streams films, TV series, music, and photos to any device, with optional hardware transcoding using the RTX 4080 Super.

**Jellyfin clients are available for:**
- Web browser
- iOS and iPad
- Android
- Apple TV and Android TV
- Roku, Fire TV
- Kodi plugin

---

## Prerequisites

- ✅ Docker and the `wcp-network` network in place
- ✅ Caddy configured with `jellyfin.wcp`
- ✅ `/mnt/media/` mounted and populated with your media

→ Follow [Part 2](../part-02-docker-tailscale/) and [Part 3](../part-03-caddy/) first.

---

## Step 1 – Organise your media

Jellyfin works best with a consistent folder structure:

```
/mnt/media/
├── films/
│   ├── The Matrix (1999)/
│   │   └── The Matrix (1999).mkv
│   └── Inception (2010)/
│       └── Inception (2010).mkv
├── series/
│   ├── Breaking Bad/
│   │   ├── Season 1/
│   │   │   ├── S01E01.mkv
│   │   │   └── S01E02.mkv
│   │   └── Season 2/
└── music/
    └── Artist Name/
        └── Album Name/
            └── 01 - Track Name.flac
```

Name films as `Title (Year)` for best metadata matching.

---

## Step 2 – Create the Jellyfin folder

```bash
mkdir -p /opt/docker/jellyfin/config
mkdir -p /opt/docker/jellyfin/cache
cd /opt/docker/jellyfin
```

---

## Step 3 – Create the compose.yml

```bash
nano compose.yml
```

```yaml
services:
  jellyfin:
    image: jellyfin/jellyfin:latest
    container_name: jellyfin
    runtime: nvidia
    environment:
      - NVIDIA_VISIBLE_DEVICES=all
      - NVIDIA_DRIVER_CAPABILITIES=all
      - TZ=Europe/Stockholm
    volumes:
      - ./config:/config
      - ./cache:/cache
      - /mnt/media:/media:ro
    networks:
      - wcp-network
    restart: unless-stopped

networks:
  wcp-network:
    external: true
```

**Key settings:**
- `runtime: nvidia` — enables GPU hardware transcoding
- `/mnt/media:/media:ro` — mounts the media library read-only inside the container

---

## Step 4 – Start Jellyfin

```bash
docker compose up -d
docker compose logs -f
```

Wait until Jellyfin is ready. Press `CTRL+C` to exit logs.

---

## Step 5 – Initial setup

Open your browser:

```
http://jellyfin.wcp
```

The setup wizard will guide you through:

1. **Create admin account** — set username and password
2. **Add media libraries:**
   - Click **Add Media Library**
   - Type: **Movies** → Path: `/media/films`
   - Type: **TV Shows** → Path: `/media/series`
   - Type: **Music** → Path: `/media/music`
3. **Choose metadata language** — English recommended for best matching
4. **Complete setup** and wait for the library scan

---

## Step 6 – Enable hardware transcoding

Hardware transcoding lets Jellyfin use the RTX 4080 Super to transcode video in real time — essential for streaming to devices that don't support your source format.

In Jellyfin:
1. Go to **Admin → Dashboard → Playback**
2. Set **Hardware acceleration** to **NVENC**
3. Enable the codec options that appear (H.264, H.265, AV1 if available)
4. Click **Save**

Test by playing a video and checking the playback info — it should show hardware transcoding active.

---

## Step 7 – Install Jellyfin on your devices

**iOS / Apple TV:** Search for **Jellyfin** in the App Store.

**Android / Android TV:** Available on Google Play and F-Droid.

**Web browser:** Any browser works — go to `http://jellyfin.wcp`.

In the app, add your server:
- Server address: `http://jellyfin.wcp` (via Tailscale)
- Log in with your admin credentials

---

## Step 8 – Remote access via Tailscale

With Tailscale connected on your device, Jellyfin is accessible from anywhere:

```
http://jellyfin.wcp
```

Or directly via Tailscale IP:

```
http://100.x.x.x:8096
```

No port forwarding required. Transcoding handles different network speeds automatically.

---

## Keeping Jellyfin updated

```bash
cd /opt/docker/jellyfin
docker compose pull
docker compose up -d
```

---

## Useful commands

```bash
# Restart Jellyfin
docker compose restart

# Follow logs
docker compose logs -f

# Trigger library rescan from CLI
docker exec jellyfin jellyfin --rescan
```

---

## What's next

Part 11 deploys Vaultwarden — a self-hosted password manager compatible with all Bitwarden clients.

**Up next:** [Part 11 – Vaultwarden](../part-11-vaultwarden/) *(coming soon)*

---

## Related guides

- [Part 3 – Caddy Reverse Proxy](../part-03-caddy/) — required for `jellyfin.wcp`
- [Tailscale – Getting Started](/guides/networking/tailscale-getting-started/) — remote access
- [Jellyfin Documentation](https://jellyfin.org/docs/) — official docs
