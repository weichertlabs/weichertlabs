---
title: WCP – Own Your Cloud
description: The WeichertLabs Cloud Platform (WCP) — a fully self-hosted cloud built on a single bare metal machine. Replace Google Drive, Netflix, ChatGPT and more with hardware you own.
date: 2026-03-31
tags: [ubuntu, self-hosted, docker, ollama, nextcloud, jellyfin, cloud, gpu]
---

This project documents building the **WeichertLabs Cloud Platform** — WCP for short — a fully self-hosted cloud environment running on a single bare metal machine.

The idea is simple: every major cloud subscription you pay for monthly can be replaced by software you run yourself, on hardware you own, with data that never leaves your home. Storage, photo backup, media streaming, AI assistant, password manager, cloud gaming — all of it, on one machine.

WCP is my personal name for this setup — a self-hosted platform that grows and evolves over time, documented here as it gets built.

---

## The hardware

Everything in this series runs on a single machine — a **Minisforum BD770i** with an RTX 4080 Super. The key design decision is **bare metal Ubuntu** rather than Proxmox with VMs. The reason: the RTX 4080 Super is the heart of the platform, shared naturally between Ollama, ComfyUI, and Sunshine via CUDA — no GPU passthrough complexity needed.

| Component | Detail | Used for |
|---|---|---|
| **CPU** | AMD Ryzen 7 7745HX | All services |
| **RAM** | 64 GB | Containers + AI models |
| **GPU** | RTX 4080 Super | Ollama, ComfyUI, Sunshine/NVENC |
| **4 TB NVMe** | Primary disk | OS, services, AI models, games |
| **4 TB HDD** | Secondary disk | Jellyfin media library |

### Disk layout

```
4TB NVMe (fast)
├── /          OS — Ubuntu
├── /opt/      All services and Docker volumes
├── /mnt/ai/   Ollama models, ComfyUI
└── /mnt/games/ Steam, Epic, GOG libraries

4TB HDD (large)
└── /mnt/media/ Jellyfin — films, series, music
```

---

## What gets built

| Service | Replaces | Purpose |
|---|---|---|
| **Nextcloud** | Google Drive / iCloud | Private cloud storage and document sync |
| **Immich** | Google Photos | Photo library with AI tagging |
| **Jellyfin** | Netflix / Plex | Local and remote media streaming |
| **Ollama + Open WebUI** | ChatGPT / Claude (offline) | Local AI assistant with full GPU acceleration |
| **ComfyUI** | Midjourney / Runway / ElevenLabs | Image, video, music and voice generation |
| **Sunshine + Moonlight** | GeForce NOW / Xbox Cloud | Cloud gaming from any device |
| **Vaultwarden** | Bitwarden cloud / 1Password | Self-hosted password manager |
| **Caddy** | nginx / Traefik | Reverse proxy and HTTPS for all services |
| **Tailscale** | VPN subscriptions | Secure remote access to everything |
| **Uptime Kuma** | — | Service monitoring and alerts |
| **Syncthing** | Dropbox / OneDrive | File sync between devices |

---

## Before you start

This series assumes Ubuntu is installed on your machine. Each service is deployed as a Docker container — if you haven't set up Docker yet, do that first:

- [Install Docker and Docker Compose on Linux](/guides/containers/docker/docker-compose-linux/) — required before deploying any service in this series
- [Tailscale – Getting Started](/guides/networking/tailscale-getting-started/) — set up remote access early, you'll want it throughout the build

### Minimum requirements to follow along

| Component | This series | Minimum |
|---|---|---|
| **OS** | Ubuntu 24.04 LTS bare metal | Any Ubuntu/Debian 20.04+ |
| **RAM** | 64 GB | 16 GB (some services will need to be skipped) |
| **Storage** | 4 TB NVMe + 2 TB HDD | 500 GB+ |
| **GPU** | RTX 4080 Super | Any NVIDIA GPU for AI acceleration (CPU works too, slower) |

Most services in this series work without a GPU — the GPU is only required for Ollama acceleration, ComfyUI, and Sunshine game streaming.

---

## Two parallel tracks

Like the Proxmox Home Lab series, WCP is documented in two ways:

**Track A – Manual** walks through every step by hand. Every config file, every command, every mistake.

**Track B – Claude Code** builds the same platform using Claude Code via SSH — testing how much an AI agent can autonomously configure a real server in 2026.

---

## Series overview

| Part | Title | Status |
|---|---|---|
| Part 1 | [Ubuntu base setup and disk layout](part-01-ubuntu-setup/) | ✅ Published |
| Part 2 | [Docker and Tailscale — the foundation](part-02-docker-tailscale/) | ✅ Published |
| Part 3 | Caddy — reverse proxy and HTTPS for all services | Coming soon |
| Part 4 | Nextcloud — your own Google Drive | Coming soon |
| Part 5 | Immich — self-hosted Google Photos | Coming soon |
| Part 6 | Ollama + Open WebUI — local AI with GPU | Coming soon |
| Part 7 | ComfyUI — creative AI studio | Coming soon |
| Part 8 | Sunshine + Moonlight — cloud gaming | Coming soon |
| Part 9 | Gaming library — Steam, Epic, GOG on Ubuntu | Coming soon |
| Part 10 | Jellyfin — your own Netflix | Coming soon |
| Part 11 | Vaultwarden — self-hosted password manager | Coming soon |
| Part 12 | Uptime Kuma + Syncthing — monitoring and sync | Coming soon |

---

{{< cards >}}
  {{< card link="manual" title="Track A — Manual" subtitle="Step by step, every config, every mistake" icon="book-open" >}}
  {{< card link="ai-assisted" title="Track B — Claude Code" subtitle="AI configures the entire stack via SSH" icon="sparkles" >}}
{{< /cards >}}
