---
title: "Part 4 – Nextcloud"
description: Deploying Nextcloud on the WCP machine using Docker Compose — your own private Google Drive with file sync, calendar, and contacts.
date: 2026-03-31
tags: [ubuntu, nextcloud, docker, self-hosted, cloud-storage]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

Nextcloud is a self-hosted file sync and collaboration platform — your own Google Drive, running on your own hardware. Files sync between all your devices, never leaving your network.

**What you get:**
- File sync across Mac, iPhone, iPad, Android, and Windows
- Calendar and contacts sync (CalDAV/CardDAV)
- Photo backup
- Document editing via OnlyOffice or Collabora (optional)
- Mobile apps for iOS and Android

---

## Prerequisites

- ✅ Docker and the `wcp-network` network in place
- ✅ Caddy running with `nextcloud.wcp` configured
- ✅ `/opt` partition with plenty of space for files

→ Follow [Part 2](/projects/weichert-cloud-platform/manual/part-02-docker-tailscale/) and [Part 3](/projects/weichert-cloud-platform/manual/part-03-caddy/) first.

---

## Step 1 – Create the Nextcloud folder

```bash
mkdir -p /opt/docker/nextcloud
cd /opt/docker/nextcloud
```

---

## Step 2 – Create the compose.yml

```bash
nano compose.yml
```

```yaml
services:
  nextcloud-db:
    image: mariadb:11
    container_name: nextcloud-db
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}
      MYSQL_DATABASE: nextcloud
      MYSQL_USER: nextcloud
      MYSQL_PASSWORD: ${DB_PASSWORD}
    volumes:
      - nextcloud-db:/var/lib/mysql
    networks:
      - wcp-network
    restart: unless-stopped

  nextcloud-redis:
    image: redis:alpine
    container_name: nextcloud-redis
    networks:
      - wcp-network
    restart: unless-stopped

  nextcloud:
    image: nextcloud:latest
    container_name: nextcloud
    depends_on:
      - nextcloud-db
      - nextcloud-redis
    environment:
      MYSQL_HOST: nextcloud-db
      MYSQL_DATABASE: nextcloud
      MYSQL_USER: nextcloud
      MYSQL_PASSWORD: ${DB_PASSWORD}
      REDIS_HOST: nextcloud-redis
      NEXTCLOUD_TRUSTED_DOMAINS: nextcloud.wcp
      NEXTCLOUD_ADMIN_USER: ${ADMIN_USER}
      NEXTCLOUD_ADMIN_PASSWORD: ${ADMIN_PASSWORD}
    volumes:
      - nextcloud-data:/var/www/html
      - /opt/nextcloud-files:/var/www/html/data
    networks:
      - wcp-network
    restart: unless-stopped

volumes:
  nextcloud-db:
  nextcloud-data:

networks:
  wcp-network:
    external: true
```

---

## Step 3 – Create the .env file

```bash
nano .env
```

```
DB_ROOT_PASSWORD=change-this-root-password
DB_PASSWORD=change-this-db-password
ADMIN_USER=admin
ADMIN_PASSWORD=change-this-admin-password
```

{{< callout type="warning" >}}
Use strong, unique passwords. Add `.env` to `.gitignore` if you version control your Docker configs — never commit passwords to a repository.
{{< /callout >}}

---

## Step 4 – Create the data directory

Nextcloud files are stored outside the container for easy backup:

```bash
sudo mkdir -p /opt/nextcloud-files
sudo chown -R www-data:www-data /opt/nextcloud-files
```

---

## Step 5 – Start Nextcloud

```bash
docker compose up -d
docker compose logs -f
```

The first start takes a minute while Nextcloud initialises. When you see `Nextcloud is ready` in the logs, press `CTRL+C`.

---

## Step 6 – Access Nextcloud

Open your browser and go to:

```
http://nextcloud.wcp
```

Log in with the admin username and password from your `.env` file.

---

## Step 7 – Performance tuning

Run these one-time setup commands inside the container:

```bash
# Enable background jobs via cron
docker exec -u www-data nextcloud php occ background:cron

# Add missing indices (speeds up the database)
docker exec -u www-data nextcloud php occ db:add-missing-indices

# Convert columns for better performance
docker exec -u www-data nextcloud php occ db:convert-filecache-bigint
```

Add a cron job on the host for background tasks:

```bash
crontab -e
```

Add:

```
*/5 * * * * docker exec -u www-data nextcloud php -f /var/www/html/cron.php
```

---

## Step 8 – Install the Nextcloud client

Install the desktop sync client on your Mac:

```bash
brew install --cask nextcloud
```

Open the app, connect to `http://nextcloud.wcp` and log in. Your files will sync automatically.

Mobile apps are available on the App Store and Google Play.

---

## Keeping Nextcloud updated

```bash
cd /opt/docker/nextcloud
docker compose pull
docker compose up -d
```

---

## Related guides

- [Part 3 – Caddy Reverse Proxy](/projects/weichert-cloud-platform/manual/part-03-caddy/) — required for `nextcloud.wcp` to work
- [Install Docker and Docker Compose on Linux](/guides/containers/docker/docker-compose-linux/) — Docker basics
- [Nextcloud Documentation](https://docs.nextcloud.com) — official docs
