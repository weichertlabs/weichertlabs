---
title: Install Home Assistant in Docker
description: How to run Home Assistant in Docker using Docker Compose on Linux — get your smart home hub up and running in minutes.
date: 2026-03-27
tags: [iot, home-assistant, docker, docker-compose, linux, smart-home]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

Home Assistant is an open-source home automation platform that lets you control and automate smart home devices locally — no cloud required. This guide covers running Home Assistant in Docker using Docker Compose on Linux.

{{< callout type="info" >}}
New to Docker? Set it up first: [Install Docker and Docker Compose on Linux](/guides/containers/docker/docker-compose-linux/)
{{< /callout >}}

## Requirements

- Linux (Ubuntu/Debian) with Docker and Docker Compose installed
- At least 2 GB RAM and 10 GB disk space
- A static IP on your Linux machine is recommended

---

## Step 1 – Create the Project Folder

```bash
mkdir -p ~/docker/homeassistant
cd ~/docker/homeassistant
```

---

## Step 2 – Create the compose.yml

```bash
nano compose.yml
```

Add the following:

```yaml
services:
  homeassistant:
    image: ghcr.io/home-assistant/home-assistant:stable
    container_name: homeassistant
    network_mode: host
    privileged: true
    environment:
      - TZ=Europe/Stockholm
    volumes:
      - ./config:/config
    restart: unless-stopped
```

**What the options mean:**
- `network_mode: host` — gives Home Assistant direct access to your network (required for device discovery)
- `privileged: true` — allows access to USB devices like Zigbee sticks
- `TZ=Europe/Stockholm` — set your timezone (change to match yours)
- `./config:/config` — stores all Home Assistant config locally in the `config` folder

---

## Step 3 – Start Home Assistant

```bash
docker compose up -d
```

Check that it started:

```bash
docker compose logs -f
```

Wait until you see something like:

```
[homeassistant] Home Assistant initialized in X.Xs
```

Press `CTRL+C` to exit the log view.

---

## Step 4 – Access the Web Interface

Open your browser and go to:

```
http://your-server-ip:8123
```

The first time you visit, Home Assistant will walk you through the initial setup — creating a user account, setting your location, and detecting devices on your network.

---

## Step 5 – Keep Home Assistant Updated

Pull the latest image and restart:

```bash
cd ~/docker/homeassistant
docker compose pull
docker compose up -d
```

---

## Useful Commands

| Command | What it does |
|---|---|
| `docker compose up -d` | Start Home Assistant |
| `docker compose down` | Stop Home Assistant |
| `docker compose restart` | Restart Home Assistant |
| `docker compose logs -f` | Follow live logs |
| `docker compose pull` | Pull latest image |

---

## Zigbee and USB Device Access

If you use a Zigbee USB stick (e.g. Sonoff Zigbee Dongle, ConBee II), find the device path:

```bash
ls /dev/ttyUSB* /dev/ttyACM*
```

Add the device to your `compose.yml`:

```yaml
services:
  homeassistant:
    image: ghcr.io/home-assistant/home-assistant:stable
    container_name: homeassistant
    network_mode: host
    privileged: true
    environment:
      - TZ=Europe/Stockholm
    volumes:
      - ./config:/config
    devices:
      - /dev/ttyUSB0:/dev/ttyUSB0
    restart: unless-stopped
```

---

## Backup Your Config

Your entire Home Assistant configuration is stored in `~/docker/homeassistant/config/`. Back it up regularly:

```bash
tar -czf homeassistant-backup-$(date +%Y%m%d).tar.gz ~/docker/homeassistant/config/
```

---

## Tips

- **Static IP** — set a static IP on your server so the Home Assistant URL never changes
- **HACS** — install the Home Assistant Community Store for additional integrations and themes
- **Reverse proxy** — use Nginx Proxy Manager or Traefik to access Home Assistant via a domain name with HTTPS
- **Home Assistant OS** — if you want the full experience with add-ons (like Zigbee2MQTT built-in), consider running Home Assistant OS on a Raspberry Pi or as a VM in Proxmox

---

## Related Links

- [Install Docker and Docker Compose on Linux](/guides/containers/docker/docker-compose-linux/) — install Docker first
- [Home Assistant Documentation](https://www.home-assistant.io/docs/) — official docs
- [HACS – Home Assistant Community Store](https://hacs.xyz) — extra integrations and themes
- [Home Assistant Community Forum](https://community.home-assistant.io) — active community support
