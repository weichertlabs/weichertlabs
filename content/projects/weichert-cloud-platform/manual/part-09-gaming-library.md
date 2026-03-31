---
title: "Part 9 – Gaming Library on Ubuntu"
description: Setting up Steam, Epic Games, GOG, and Ubisoft on Ubuntu with Proton for Windows game compatibility — your complete gaming library on Linux.
date: 2026-03-31
tags: [ubuntu, gaming, steam, proton, epic, gog, linux, gpu]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

Linux gaming has come a long way. With Proton, most Windows games run on Ubuntu without any configuration — just install and play. This part sets up the complete gaming stack on the WCP machine, ready to be streamed via Sunshine to any device.

---

## Prerequisites

- ✅ Ubuntu 24.04 LTS with NVIDIA drivers
- ✅ `/mnt/games/` directory created with sufficient space
- ✅ Sunshine installed (optional — see [Part 8](../part-08-sunshine-moonlight/))

---

## Step 1 – Enable 32-bit architecture

Many games require 32-bit libraries:

```bash
sudo dpkg --add-architecture i386
sudo apt update
```

---

## Step 2 – Install Steam

Steam is available directly from the Ubuntu repositories:

```bash
sudo apt install -y steam-installer
```

Or download the `.deb` from [store.steampowered.com](https://store.steampowered.com/about/):

```bash
wget https://cdn.cloudflare.steamstatic.com/client/installer/steam.deb
sudo apt install -y ./steam.deb
```

Launch Steam and log in with your account.

---

## Step 3 – Enable Proton for Windows games

Proton lets you run Windows-only games on Linux with minimal setup.

In Steam:
1. Go to **Steam → Settings → Compatibility**
2. Enable **"Enable Steam Play for all other titles"**
3. Select **Proton Experimental** (or the latest stable version)
4. Click **OK** and restart Steam

Most games will now install and run via Proton automatically. Check [ProtonDB](https://www.protondb.com) for compatibility ratings before buying.

---

## Step 4 – Move Steam library to /mnt/games

By default Steam stores games in `~/.steam`. Move the library to the dedicated games partition:

1. In Steam: **Settings → Downloads → Steam Library Folders**
2. Click **Add Library Folder**
3. Select `/mnt/games/steam`
4. Set it as the default library

```bash
# Create the directory first
mkdir -p /mnt/games/steam
```

---

## Step 5 – Install Heroic Games Launcher (Epic and GOG)

Heroic is an open-source launcher for Epic Games and GOG:

```bash
# Download latest AppImage from GitHub
wget https://github.com/Heroic-Games-Launcher/HeroicGamesLauncher/releases/latest/download/heroic-x86_64.AppImage
chmod +x heroic-x86_64.AppImage

# Move to a permanent location
mkdir -p ~/.local/bin
mv heroic-x86_64.AppImage ~/.local/bin/heroic

# Run it
~/.local/bin/heroic
```

Or install via Flatpak:

```bash
sudo apt install -y flatpak
flatpak install flathub com.heroicgameslauncher.hgl
```

In Heroic:
1. Log in to your Epic Games and/or GOG accounts
2. Set the game installation path to `/mnt/games/epic` and `/mnt/games/gog`
3. Configure Wine/Proton version under **Settings → Game Defaults**

---

## Step 6 – Ubisoft Connect (via Heroic or Lutris)

Ubisoft Connect requires extra setup on Linux.

**Option A – Via Heroic:**
Some Ubisoft games purchased on Epic can be launched through Heroic with Ubisoft Connect running inside the Wine environment.

**Option B – Via Lutris:**

```bash
sudo apt install -y lutris
```

Search for your Ubisoft games on [lutris.net](https://lutris.net) — most have community-maintained install scripts that handle the Ubisoft Connect setup automatically.

---

## Step 7 – Optimise for gaming performance

### Set NVIDIA power mode to maximum performance

```bash
sudo nvidia-smi --persistence-mode=1
sudo nvidia-smi -pm 1
```

Add to `/etc/rc.local` to apply on boot.

### Install GameMode

GameMode optimises system performance when a game is running:

```bash
sudo apt install -y gamemode
```

In Steam, add `gamemoderun %command%` to a game's launch options for better performance.

### Install MangoHud (performance overlay)

MangoHud shows FPS, GPU usage, temperatures and more as an overlay:

```bash
sudo apt install -y mangohud
```

Add `mangohud %command%` to Steam launch options to enable it per game.

---

## ProtonDB — check game compatibility

Before installing a Windows game, check [protondb.com](https://www.protondb.com) for community reports. Games are rated:

| Rating | Meaning |
|---|---|
| **Platinum** | Works perfectly out of the box |
| **Gold** | Works with minor tweaks |
| **Silver** | Works but with some issues |
| **Bronze** | Runs but significant problems |
| **Borked** | Does not work |

The vast majority of popular games are Gold or Platinum.

---

## Useful commands

```bash
# Check GPU temperature while gaming
nvidia-smi --query-gpu=temperature.gpu --format=csv,noheader -l 1

# Monitor GPU usage
watch -n 1 nvidia-smi

# List installed Proton versions
ls ~/.steam/steam/steamapps/common/ | grep Proton
```

---

## What's next

Part 10 deploys Jellyfin — your own Netflix, streaming your media library to any device.

**Up next:** [Part 10 – Jellyfin](../part-10-jellyfin/) *(coming soon)*

---

## Related links

- [Part 8 – Sunshine and Moonlight](../part-08-sunshine-moonlight/) — stream these games to any device
- [ProtonDB](https://www.protondb.com) — community game compatibility reports
- [Heroic Games Launcher](https://heroicgameslauncher.com) — Epic and GOG on Linux
- [Lutris](https://lutris.net) — game manager for Linux
