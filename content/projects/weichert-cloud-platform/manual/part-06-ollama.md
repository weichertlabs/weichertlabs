---
title: "Part 6 – Ollama and Open WebUI"
description: Deploying Ollama with GPU acceleration and Open WebUI on the WCP machine — a fully local AI assistant running on your own RTX 4080 Super.
date: 2026-03-31
tags: [ubuntu, ollama, open-webui, docker, ai, nvidia, gpu, local-ai]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

Ollama runs large language models locally on your own hardware. Combined with Open WebUI, you get a ChatGPT-like interface running entirely on your machine — no internet required, no API costs, no data leaving your home.

With an RTX 4080 Super and 16 GB VRAM, models respond in seconds rather than minutes.

---

## Prerequisites

- ✅ Docker with NVIDIA Container Toolkit installed
- ✅ NVIDIA drivers running (`nvidia-smi` works)
- ✅ Caddy configured with `ai.wcp`
- ✅ The `wcp-network` Docker network created

→ Follow [Part 2](../part-02-docker-tailscale/) first for Docker and NVIDIA Container Toolkit setup.

---

## Step 1 – Create the folder

```bash
mkdir -p /opt/docker/ollama
cd /opt/docker/ollama
```

---

## Step 2 – Create the compose.yml

```bash
nano compose.yml
```

```yaml
services:
  ollama:
    image: ollama/ollama:latest
    container_name: ollama
    runtime: nvidia
    environment:
      - NVIDIA_VISIBLE_DEVICES=all
    volumes:
      - /mnt/ai/ollama:/root/.ollama
    networks:
      - wcp-network
    restart: unless-stopped

  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    depends_on:
      - ollama
    environment:
      - OLLAMA_BASE_URL=http://ollama:11434
    volumes:
      - open-webui-data:/app/backend/data
    networks:
      - wcp-network
    restart: unless-stopped

volumes:
  open-webui-data:

networks:
  wcp-network:
    external: true
```

**Key settings:**
- `runtime: nvidia` — enables GPU access for the Ollama container
- `/mnt/ai/ollama:/root/.ollama` — stores models on the NVMe disk outside the container
- `OLLAMA_BASE_URL` — tells Open WebUI where to find Ollama

---

## Step 3 – Start the services

```bash
docker compose up -d
docker compose logs -f
```

Wait until both containers are running. Press `CTRL+C` to exit logs.

---

## Step 4 – Pull your first model

```bash
docker exec ollama ollama pull llama3.1
```

This downloads the model to `/mnt/ai/ollama`. With an RTX 4080 Super and 16 GB VRAM, `llama3.1:8b` runs entirely on the GPU for fast responses.

Other recommended models:

```bash
# Fast and capable
docker exec ollama ollama pull mistral

# Great for coding
docker exec ollama ollama pull codellama

# Lightweight, very fast
docker exec ollama ollama pull phi3

# Strong reasoning
docker exec ollama ollama pull deepseek-r1
```

---

## Step 5 – Access Open WebUI

Open your browser:

```
http://ai.wcp
```

Create an admin account on the first visit. Open WebUI connects to Ollama automatically and shows all downloaded models.

---

## Step 6 – Verify GPU is being used

While a model is running, check GPU usage:

```bash
nvidia-smi
```

You should see memory allocated to the Ollama process and GPU utilization above 0%.

---

## Model recommendations for RTX 4080 Super

With 16 GB VRAM you can run large models comfortably:

| Model | VRAM needed | Best for |
|---|---|---|
| `phi3` | ~2 GB | Fast answers, low resource |
| `llama3.2:3b` | ~3 GB | Quick everyday use |
| `mistral:7b` | ~5 GB | Balanced speed and quality |
| `llama3.1:8b` | ~6 GB | Great general purpose |
| `codellama:13b` | ~10 GB | Code generation |
| `llama3.1:13b` | ~10 GB | High quality responses |

---

## Useful commands

```bash
# List downloaded models
docker exec ollama ollama list

# Pull a new model
docker exec ollama ollama pull modelname

# Remove a model
docker exec ollama ollama rm modelname

# Chat directly in terminal
docker exec -it ollama ollama run llama3.1
```

---

## What's next

Part 7 deploys ComfyUI — a node-based creative AI studio for image, video, and music generation, also using the RTX 4080 Super.

**Up next:** [Part 7 – ComfyUI](../part-07-comfyui/) *(coming soon)*

---

## Related guides

- [Ollama – Run AI Models Locally](/guides/ai/ollama-local-ai/) — standalone Ollama guide
- [Ollama on Linux with NVIDIA GPU](/guides/ai/ollama-nvidia-gpu-linux/) — GPU setup details
- [Part 2 – Docker and Tailscale](../part-02-docker-tailscale/) — Docker and GPU prerequisites
