---
title: Ollama – Run AI Models Locally
description: How to install Ollama and run large language models locally on Linux or macOS — no cloud, no API keys, full privacy.
date: 2026-03-27
tags: [ai, ollama, local-ai, llm, linux, macos]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

Ollama makes it easy to run large language models (LLMs) locally on your own hardware — no internet required, no API keys, no data sent to the cloud. Just download a model and start chatting.

This guide covers installation on Linux and macOS, running your first model, and useful commands for managing models.

## Requirements

- Linux (Ubuntu/Debian) or macOS
- Minimum 8 GB RAM (16 GB+ recommended)
- For GPU acceleration: NVIDIA GPU with CUDA support (Linux) or Apple Silicon Mac

---

## Install Ollama

**Linux:**

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

The installer automatically detects your GPU and sets up CUDA support if available.

**macOS:**

```bash
brew install ollama
```

Or download the app directly from [ollama.com](https://ollama.com).

---

## Start Ollama

**Linux** – Ollama runs as a systemd service after installation:

```bash
sudo systemctl start ollama
sudo systemctl enable ollama  # start on boot
```

**macOS** – Start from terminal:

```bash
ollama serve
```

Or just open the Ollama app from Applications.

---

## Run Your First Model

Pull and run a model in one command:

```bash
ollama run llama3.2
```

The first run downloads the model (a few GB depending on the model). After that it loads from local storage — no internet needed.

You're now in an interactive chat. Type your message and press Enter. Type `/bye` to exit.

---

## Popular Models

| Model | Size | Good for |
|---|---|---|
| `llama3.2` | 2B / 3B | Fast, general purpose, low RAM |
| `llama3.1` | 8B | Good balance of speed and quality |
| `mistral` | 7B | Fast, great for coding and reasoning |
| `codellama` | 7B | Code generation and explanation |
| `phi3` | 3.8B | Microsoft model, very fast |
| `deepseek-r1` | 7B | Strong reasoning and math |
| `gemma2` | 9B | Google model, good quality |

Browse all models at [ollama.com/library](https://ollama.com/library).

---

## Useful Commands

| Command | What it does |
|---|---|
| `ollama run llama3.2` | Pull and run a model |
| `ollama pull llama3.2` | Download a model without running it |
| `ollama list` | List all downloaded models |
| `ollama rm llama3.2` | Delete a model |
| `ollama show llama3.2` | Show model details |
| `ollama ps` | Show currently running models |
| `ollama serve` | Start the Ollama server manually |

---

## Run a Model Non-Interactively

Send a single prompt from the terminal:

```bash
ollama run llama3.2 "Explain what Docker is in simple terms"
```

Pipe input to a model:

```bash
cat script.sh | ollama run codellama "Review this bash script for errors"
```

---

## Ollama REST API

Ollama exposes a local REST API on port 11434 — useful for integrating with other tools:

```bash
curl http://localhost:11434/api/generate -d '{
  "model": "llama3.2",
  "prompt": "What is Proxmox?",
  "stream": false
}'
```

This API is compatible with the OpenAI API format, making it easy to use Ollama as a drop-in replacement in apps that support OpenAI.

---

## Add a Web Interface with Open WebUI

For a ChatGPT-like interface in your browser, run Open WebUI alongside Ollama:

```bash
docker run -d \
  --network=host \
  -v open-webui:/app/backend/data \
  --name open-webui \
  --restart always \
  ghcr.io/open-webui/open-webui:main
```

Open your browser at `http://localhost:8080` and connect to your local Ollama instance.

---

## Tips

- **RAM usage** — each model loads fully into RAM (or VRAM with GPU). Make sure you have enough before pulling large models.
- **GPU acceleration** — if you have an NVIDIA GPU, Ollama uses it automatically after installation. Speeds up inference significantly.
- **Apple Silicon** — Ollama uses Metal on Apple Silicon Macs for GPU acceleration out of the box.
- **Model storage** — models are stored in `~/.ollama/models` on Linux and macOS.

---

## Related Links

- [Ollama Official Site](https://ollama.com) — model library and documentation
- [Open WebUI](https://github.com/open-webui/open-webui) — web interface for Ollama
- [Ollama on Linux with NVIDIA GPU](/guides/ai/ollama-nvidia-gpu-linux/) — maximize performance with GPU acceleration
- [Install Docker and Docker Compose on Linux](/guides/containers/docker/docker-compose-linux/) — required for Open WebUI
