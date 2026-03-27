---
title: Install Docker and Docker Compose on Linux
description: How to install Docker Engine and Docker Compose on Ubuntu or Debian, fix permissions so you don't need sudo, and organize your containers with a clean folder structure.
date: 2026-03-27
tags: [docker, docker-compose, linux, ubuntu, debian, containers]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

Docker lets you run applications in isolated containers. Docker Compose lets you manage multiple containers at once — for example a web app, a database, and a cache — using a single file and a single command.

This guide covers installation on Ubuntu and Debian, fixing permissions so you don't need to type `sudo` every time, and setting up a clean folder structure for your containers.

## Requirements

- Ubuntu 20.04+ or Debian 11+
- A user account with sudo privileges
- An internet connection

---

## Step 1 – Install Docker Engine

Add Docker's official repository and install:

```bash
# Update and install dependencies
sudo apt update
sudo apt install -y ca-certificates curl gnupg

# Add Docker's official GPG key
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
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

{{< callout type="info" >}}
On **Debian**, replace `ubuntu` with `debian` in the repository URL above.
{{< /callout >}}

---

## Step 2 – Verify the Installation

```bash
docker --version
docker compose version
```

You should see something like:

```
Docker version 26.x.x
Docker Compose version v2.x.x
```

---

## Step 3 – Fix Permissions (No More sudo)

By default, Docker requires root privileges. Add your user to the `docker` group to fix this:

```bash
sudo usermod -aG docker $USER
newgrp docker
```

Verify it works without sudo:

```bash
docker run hello-world
```

If you see a welcome message without any errors — you're all set.

{{< callout type="warning" >}}
Adding your user to the docker group gives that user root-equivalent access to the system. This is fine for a lab environment, but keep it in mind for production systems.
{{< /callout >}}

---

## Step 4 – Organize Your Containers

A clean folder structure makes managing many containers much easier, especially as your lab grows.

Recommended structure:

```
~/docker/
├── nginx/
│   ├── compose.yml
│   └── config/
├── nextcloud/
│   ├── compose.yml
│   └── data/
├── monitoring/
│   ├── compose.yml
│   └── prometheus.yml
└── databases/
    ├── compose.yml
    └── init/
```

Each service gets its own folder with its own `compose.yml`. This means you can start, stop, and manage each service independently without affecting the others.

---

## Step 5 – Example compose.yml

Here is a simple example running Nginx:

```yaml
# ~/docker/nginx/compose.yml

services:
  nginx:
    image: nginx:latest
    container_name: my-nginx
    ports:
      - "8080:80"
    volumes:
      - ./config/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./html:/usr/share/nginx/html:ro
    restart: unless-stopped
```

Start the service:

```bash
cd ~/docker/nginx
docker compose up -d
```

---

## Useful Commands

| Command | What it does |
|---|---|
| `docker compose up -d` | Start containers in the background |
| `docker compose down` | Stop and remove containers |
| `docker compose restart` | Restart containers |
| `docker compose logs -f` | Follow live logs |
| `docker compose pull` | Pull the latest image |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers including stopped |
| `docker stats` | Real-time CPU and memory usage |
| `docker system prune` | Clean up unused resources |

---

## Tips

**Use named volumes for persistent data:**
```yaml
volumes:
  db-data:

services:
  postgres:
    image: postgres:16
    volumes:
      - db-data:/var/lib/postgresql/data
```

**Store sensitive values in a `.env` file:**
```bash
# ~/docker/databases/.env
POSTGRES_PASSWORD=yourpassword
POSTGRES_USER=admin
```

```yaml
# compose.yml reads .env automatically
services:
  postgres:
    environment:
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_USER: ${POSTGRES_USER}
```

Add `.env` to `.gitignore` if you use Git — never push passwords to a repository.

---

## Related Links

- [Docker Documentation](https://docs.docker.com) — official Docker docs
- [Docker Compose Reference](https://docs.docker.com/compose/compose-file/) — full compose.yml reference
- [Docker Hub](https://hub.docker.com) — browse official container images
