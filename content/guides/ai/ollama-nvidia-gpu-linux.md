---
title: Ollama on Linux with NVIDIA GPU
description: How to set up Ollama with NVIDIA GPU acceleration on Linux using CUDA — run large AI models significantly faster on your own hardware.
date: 2026-03-27
tags: [ai, ollama, nvidia, cuda, gpu, linux, local-ai]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

Running Ollama with a NVIDIA GPU dramatically speeds up AI model inference. Instead of waiting 30+ seconds for a response on CPU, a decent GPU can respond in seconds. This guide covers setting up CUDA drivers and verifying GPU acceleration with Ollama on Linux.

## Requirements

- Linux (Ubuntu 20.04+ or Debian 11+)
- NVIDIA GPU with CUDA support (GTX 900 series or newer)
- Ollama installed (see [Ollama – Run AI Models Locally](/guides/ai/ollama-local-ai/))

---

## Step 1 – Check Your GPU

Verify your NVIDIA GPU is detected:

```bash
lspci | grep -i nvidia
```

Check if NVIDIA drivers are already installed:

```bash
nvidia-smi
```

If `nvidia-smi` works and shows your GPU — skip to Step 3.

---

## Step 2 – Install NVIDIA Drivers

Add the NVIDIA driver repository and install:

```bash
sudo apt update
sudo apt install -y ubuntu-drivers-common
sudo ubuntu-drivers autoinstall
```

Or install a specific driver version:

```bash
sudo apt install -y nvidia-driver-535
```

Reboot after installation:

```bash
sudo reboot
```

Verify after reboot:

```bash
nvidia-smi
```

Example output:

```
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 535.x    Driver Version: 535.x    CUDA Version: 12.x            |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence  | Bus-Id        Disp.A | Volatile Uncorr. ECC |
|   0  RTX 4080 Super      Off  | 00000000:01:00.0 Off |                  N/A |
+-----------------------------------------------------------------------------+
```

---

## Step 3 – Install CUDA Toolkit

Ollama's installer handles CUDA automatically, but installing the toolkit gives you additional tools:

```bash
sudo apt install -y nvidia-cuda-toolkit
```

Verify CUDA:

```bash
nvcc --version
```

---

## Step 4 – Install or Reinstall Ollama

If you already have Ollama installed, the installer will update it and detect your GPU:

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

The installer automatically:
- Detects your NVIDIA GPU
- Configures CUDA support
- Sets up the systemd service

---

## Step 5 – Verify GPU Acceleration

Start Ollama and run a model:

```bash
sudo systemctl restart ollama
ollama run llama3.1
```

While the model is running, open a second terminal and check GPU usage:

```bash
nvidia-smi
```

You should see GPU memory being used and GPU utilization above 0%. This confirms Ollama is using your GPU.

---

## Choosing Models for Your GPU

GPU VRAM determines which models you can run efficiently:

| VRAM | Recommended models |
|---|---|
| 4 GB | `llama3.2:3b`, `phi3`, `gemma2:2b` |
| 8 GB | `llama3.1:8b`, `mistral:7b`, `codellama:7b` |
| 12 GB | `llama3.1:8b` comfortably, some 13B models |
| 16 GB+ | `llama3.1:13b`, larger models |
| 24 GB (RTX 4080 Super) | `llama3.1:13b`, `codellama:34b`, most 30B models |

With an RTX 4080 Super and 16 GB VRAM you can run very capable models at full GPU speed.

---

## Monitor GPU Usage

Watch GPU usage in real time while running models:

```bash
watch -n 1 nvidia-smi
```

Or use `nvtop` for a more detailed view:

```bash
sudo apt install -y nvtop
nvtop
```

---

## Troubleshooting

**Ollama not using GPU:**

Check Ollama logs for GPU detection:

```bash
sudo journalctl -u ollama -f
```

Look for lines mentioning CUDA or your GPU model. If not found, reinstall Ollama after installing NVIDIA drivers.

**Out of VRAM:**

If a model is too large for your VRAM, Ollama automatically offloads layers to CPU RAM. Performance drops significantly but it still works. Use a smaller model or one with fewer parameters.

---

## Related Links

- [Ollama – Run AI Models Locally](/guides/ai/ollama-local-ai/) — basic Ollama setup guide
- [NVIDIA CUDA Documentation](https://docs.nvidia.com/cuda/) — official CUDA docs
- [Ollama GPU Documentation](https://github.com/ollama/ollama/blob/main/docs/gpu.md) — Ollama GPU support details
