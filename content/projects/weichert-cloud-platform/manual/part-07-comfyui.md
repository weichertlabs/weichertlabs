---
title: "Part 7 – ComfyUI"
description: Deploying ComfyUI on the WCP machine — a node-based creative AI studio for image generation, video, music, and voice synthesis, running on the RTX 4080 Super.
date: 2026-03-31
tags: [ubuntu, comfyui, docker, ai, nvidia, gpu, stable-diffusion, creative]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

ComfyUI is a node-based interface for running AI creative models — image generation with Stable Diffusion and FLUX, video generation, music with ACE-Step, and voice synthesis with Kokoro TTS. Everything runs locally on the RTX 4080 Super.

This is the creative studio layer of the WCP — used for generating images, videos, music, and voice for projects including the WeichertLabs YouTube pipeline.

---

## Prerequisites

- ✅ Docker with NVIDIA Container Toolkit installed
- ✅ NVIDIA drivers running
- ✅ Caddy configured (optional — ComfyUI can also run on a direct port)
- ✅ `/mnt/ai/` directory created

→ Follow [Part 2](../part-02-docker-tailscale/) for Docker and GPU setup.

---

## Step 1 – Create the folder

```bash
mkdir -p /opt/docker/comfyui
mkdir -p /mnt/ai/comfyui/models
mkdir -p /mnt/ai/comfyui/output
mkdir -p /mnt/ai/comfyui/input
cd /opt/docker/comfyui
```

---

## Step 2 – Create the compose.yml

```bash
nano compose.yml
```

```yaml
services:
  comfyui:
    image: ghcr.io/ai-dock/comfyui:latest-cuda
    container_name: comfyui
    runtime: nvidia
    environment:
      - NVIDIA_VISIBLE_DEVICES=all
      - NVIDIA_DRIVER_CAPABILITIES=all
    ports:
      - "8188:8188"
    volumes:
      - /mnt/ai/comfyui/models:/opt/ComfyUI/models
      - /mnt/ai/comfyui/output:/opt/ComfyUI/output
      - /mnt/ai/comfyui/input:/opt/ComfyUI/input
    networks:
      - wcp-network
    restart: unless-stopped

networks:
  wcp-network:
    external: true
```

**Note:** ComfyUI is accessed directly on port 8188 rather than through Caddy — it works better without a reverse proxy for the websocket connections it uses.

---

## Step 3 – Start ComfyUI

```bash
docker compose up -d
docker compose logs -f
```

The first start downloads the ComfyUI image — this is a large download. Wait until you see the server is ready. Press `CTRL+C` to exit logs.

---

## Step 4 – Access ComfyUI

Via Tailscale:

```
http://100.x.x.x:8188
```

Or if you're on the same local network:

```
http://192.168.1.50:8188
```

You'll see the ComfyUI node editor interface.

---

## Step 5 – Download models

Models are stored in `/mnt/ai/comfyui/models/` — organised by type.

### Image generation — FLUX (recommended)

FLUX is the current state-of-the-art for image generation. Download from [Hugging Face](https://huggingface.co/black-forest-labs/FLUX.1-dev):

```bash
# Place in:
/mnt/ai/comfyui/models/unet/
```

### Image generation — Stable Diffusion

SDXL and SD 1.5 models are widely available on [Civitai](https://civitai.com) and Hugging Face:

```bash
# Place in:
/mnt/ai/comfyui/models/checkpoints/
```

### Music generation — ACE-Step

ACE-Step generates music from text prompts and is licensed under Apache 2.0 — safe for commercial use:

```bash
# Download from Hugging Face and place in:
/mnt/ai/comfyui/models/ace-step/
```

### Voice/TTS — Kokoro

Kokoro TTS generates natural-sounding speech from text:

```bash
# Place in:
/mnt/ai/comfyui/models/kokoro/
```

---

## Step 6 – Install custom nodes

ComfyUI's power comes from its custom node ecosystem. Install via the ComfyUI Manager:

1. In the ComfyUI interface, look for the **Manager** button
2. If not present, install ComfyUI Manager manually:

```bash
docker exec -it comfyui bash
cd /opt/ComfyUI/custom_nodes
git clone https://github.com/ltdrdata/ComfyUI-Manager.git
exit
docker compose restart
```

From the Manager you can browse and install nodes for video generation, ControlNet, upscaling, and more.

---

## GPU sharing with Ollama

Both Ollama and ComfyUI use the RTX 4080 Super. They handle GPU sharing gracefully — when ComfyUI is generating, Ollama's responses may be slightly slower, and vice versa. In practice on 16 GB VRAM this rarely causes issues unless both are under heavy load simultaneously.

Monitor GPU usage:

```bash
watch -n 1 nvidia-smi
```

---

## WeichertLabs YouTube pipeline

ComfyUI is central to the WeichertLabs content pipeline:

| Tool | Purpose |
|---|---|
| **ComfyUI + ACE-Step** | Background music generation (Apache 2.0 — commercial OK) |
| **ComfyUI + Kokoro TTS** | Voice narration for videos |
| **Whisper** | Auto-generate subtitles/SRT from the narration |
| **DaVinci Resolve** | Final video editing |

The entire pipeline runs locally — no subscriptions, no data sent externally.

---

## Useful commands

```bash
# Restart ComfyUI
docker compose restart

# Follow logs
docker compose logs -f

# Check GPU usage while generating
nvidia-smi
```

---

## What's next

Part 8 sets up Sunshine and Moonlight for cloud gaming — streaming games from the RTX 4080 Super to any device.

**Up next:** [Part 8 – Sunshine and Moonlight](../part-08-sunshine-moonlight/) *(coming soon)*

---

## Related guides

- [Ollama on Linux with NVIDIA GPU](/guides/ai/ollama-nvidia-gpu-linux/) — GPU setup details
- [Part 6 – Ollama and Open WebUI](../part-06-ollama/) — local AI assistant running alongside ComfyUI
- [ComfyUI GitHub](https://github.com/comfyanonymous/ComfyUI) — official repo and documentation
