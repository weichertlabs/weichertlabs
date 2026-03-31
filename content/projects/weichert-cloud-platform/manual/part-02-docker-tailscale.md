---
title: "Part 2 – Docker and Tailscale"
description: Setting up Docker and Tailscale on the WCP machine — the two foundations that every service in this series depends on.
date: 2026-03-31
tags: [ubuntu, docker, tailscale, networking, self-hosted]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

Before deploying any service, two things need to be in place — Docker for running containers, and Tailscale for secure remote access. This part sets up both.

---

## Prerequisites

- ✅ Ubuntu 24.04 LTS installed and updated
- ✅ NVIDIA drivers installed and `nvidia-smi` working
- ✅ SSH access to the machine

→ Follow [Part 1 – Ubuntu Base Setup and Disk Layout](../part-01-ubuntu-setup/) first if you haven't already.

---

## Part A – Docker

Every service in WCP runs as a Docker container. Docker keeps services isolated, easy to update, and simple to manage.

### Step 1 – Install Docker Engine

```bash
# Install dependencies
sudo apt update
sudo apt install -y ca-certificates curl gnupg

# Add Docker's GPG key
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add Docker's repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin
```

### Step 2 – Fix Permissions

Add your user to the docker group so you don't need sudo every time:

```bash
sudo usermod -aG docker $USER
newgrp docker
```

Verify:

```bash
docker run hello-world
```

### Step 3 – Install NVIDIA Container Toolkit

This allows Docker containers to use the GPU — required for Ollama and ComfyUI:

```bash
# Add NVIDIA container toolkit repository
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | \
  sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
  sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

# Install
sudo apt update
sudo apt install -y nvidia-container-toolkit

# Configure Docker to use NVIDIA runtime
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```

Verify GPU is available in Docker:

```bash
docker run --rm --gpus all nvidia/cuda:12.0-base-ubuntu22.04 nvidia-smi
```

You should see your RTX 4080 Super in the output.

### Step 4 – Set Up the Docker Directory Structure

All services live under `/opt/docker` — one folder per service:

```bash
sudo mkdir -p /opt/docker
sudo chown -R $USER:$USER /opt/docker
```

Each service gets its own folder:

```
/opt/docker/
├── nextcloud/
│   └── compose.yml
├── jellyfin/
│   └── compose.yml
├── ollama/
│   └── compose.yml
├── caddy/
│   └── compose.yml
└── ...
```

### Step 5 – Create a Shared Docker Network

Some services need to communicate with each other — especially services behind Caddy. Create a shared network:

```bash
docker network create wcp-network
```

This network will be referenced in each service's `compose.yml` throughout the series.

---

## Part B – Tailscale

Tailscale provides secure remote access to the WCP machine from anywhere — your Mac, phone, or any other device. No port forwarding, no exposed ports.

Setting up Tailscale now means you can manage the server remotely for everything that follows in this series.

### Step 1 – Install Tailscale

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

### Step 2 – Start and Authenticate

```bash
sudo tailscale up
```

Open the URL shown in the terminal, log in with your Tailscale account, and authorize the device.

### Step 3 – Verify

```bash
tailscale status
tailscale ip
```

Note the Tailscale IP — you can now SSH to the machine using this IP from anywhere:

```bash
ssh patrik@100.x.x.x
```

### Step 4 – Enable on Boot

```bash
sudo systemctl enable tailscaled
```

### Step 5 – Disable Key Expiry

In the Tailscale admin console at [login.tailscale.com](https://login.tailscale.com):

1. Find your WCP machine
2. Click the three dots → **Disable key expiry**

This ensures the machine stays connected permanently without needing to re-authenticate.

### Step 6 – Enable MagicDNS (Recommended)

In the Tailscale admin console under **DNS → Enable MagicDNS**.

Now you can reach the machine by hostname instead of IP:

```bash
ssh patrik@wcp
```

---

## Useful aliases for the WCP machine

Add these to your `~/.bashrc` on the WCP machine to make daily management faster:

```bash
# Docker shortcuts
alias dps="docker ps"
alias dpsa="docker ps -a"
alias dcu="docker compose up -d"
alias dcd="docker compose down"
alias dcl="docker compose logs -f"
alias dcr="docker compose restart"
alias dcp="docker compose pull && docker compose up -d"

# Navigate to services quickly
alias wcp="cd /opt/docker"
alias cdnc="cd /opt/docker/nextcloud"
alias cdjf="cd /opt/docker/jellyfin"
alias cdol="cd /opt/docker/ollama"

# System
alias update="sudo apt update && sudo apt upgrade -y"
alias ports="ss -tulnp"
alias myip="tailscale ip"
```

Reload:

```bash
source ~/.bashrc
```

---

## Verification checklist

Before moving on to Part 3, verify:

```bash
# Docker works without sudo
docker ps

# GPU available in Docker
docker run --rm --gpus all nvidia/cuda:12.0-base-ubuntu22.04 nvidia-smi

# Tailscale connected
tailscale status

# Shared network exists
docker network ls | grep wcp-network

# Directory structure in place
ls /opt/docker
```

---

## What's next

With Docker and Tailscale in place, Part 3 sets up Caddy as a reverse proxy — giving every service a clean URL with automatic HTTPS instead of having to remember port numbers.

**Up next:** Part 3 – Caddy Reverse Proxy *(coming soon)*

---

## Related guides

- [Install Docker and Docker Compose on Linux](/guides/containers/docker/docker-compose-linux/) — more detail on Docker setup
- [Tailscale – Getting Started](/guides/networking/tailscale-getting-started/) — Tailscale basics
- [Ollama on Linux with NVIDIA GPU](/guides/ai/ollama-nvidia-gpu-linux/) — GPU acceleration for AI models
