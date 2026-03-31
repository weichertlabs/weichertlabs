---
title: "Part 8 – Sunshine and Moonlight"
description: Setting up Sunshine game streaming server on the WCP machine and connecting with Moonlight — cloud gaming from any device using the RTX 4080 Super.
date: 2026-03-31
tags: [ubuntu, sunshine, moonlight, gaming, streaming, nvidia, gpu]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

Sunshine is an open-source game streaming server that uses NVENC on the RTX 4080 Super to stream games to any device running Moonlight. The result is a personal GeForce NOW — low latency, high quality, running on your own hardware.

**Moonlight clients are available for:**
- iOS and iPad
- Android
- macOS
- Windows
- Linux
- Some smart TVs

---

## Prerequisites

- ✅ Ubuntu bare metal with NVIDIA drivers installed
- ✅ Games installed (Steam, Epic, GOG — see Part 9)
- ✅ Tailscale connected for remote access

{{< callout type="info" >}}
Sunshine works best on bare metal — not in a Docker container or VM. GPU passthrough adds latency. This is one of the reasons WCP runs bare metal Ubuntu instead of Proxmox.
{{< /callout >}}

---

## Step 1 – Install Sunshine

Download the latest `.deb` package from the [Sunshine GitHub releases page](https://github.com/LizardByte/Sunshine/releases):

```bash
# Download (replace with latest version)
wget https://github.com/LizardByte/Sunshine/releases/latest/download/sunshine-ubuntu-24.04-amd64.deb

# Install
sudo apt install -y ./sunshine-ubuntu-24.04-amd64.deb
```

---

## Step 2 – Configure Sunshine to start on login

Sunshine needs a display server running. Since this is a headless server, set up a virtual display:

```bash
# Install a virtual display (dummy HDMI)
sudo apt install -y xserver-xorg-video-dummy x11-utils

# Create the Xorg config for a virtual display
sudo nano /etc/X11/xorg.conf.d/10-dummy.conf
```

Add:

```
Section "Device"
    Identifier "Configured Video Device"
    Driver "dummy"
    VideoRam 256000
EndSection

Section "Monitor"
    Identifier "Configured Monitor"
    HorizSync 5.0 - 1000.0
    VertRefresh 5.0 - 200.0
    Modeline "1920x1080_60" 172.80 1920 2040 2248 2576 1080 1081 1084 1118 -HSync +Vsync
EndSection

Section "Screen"
    Identifier "Default Screen"
    Monitor "Configured Monitor"
    Device "Configured Video Device"
    DefaultDepth 24
    SubSection "Display"
        Depth 24
        Modes "1920x1080_60"
    EndSubSection
EndSection
```

---

## Step 3 – Enable and start Sunshine

```bash
# Enable Sunshine as a user service
systemctl --user enable sunshine
systemctl --user start sunshine
```

---

## Step 4 – Access the Sunshine web UI

Sunshine has a web interface for configuration:

```
https://localhost:47990
```

Or via Tailscale:

```
https://100.x.x.x:47990
```

Accept the self-signed certificate warning. Log in with the default credentials and change them immediately:

- Default username: `admin`
- Default password: `admin`

---

## Step 5 – Add applications to Sunshine

In the Sunshine web UI, go to **Configuration → Applications** and add your games and launchers:

**Steam:**
- Name: `Steam`
- Command: `steam`

**Steam Big Picture (recommended for controller use):**
- Name: `Steam Big Picture`
- Command: `steam steam://open/bigpicture`

**Desktop:**
- Name: `Desktop`
- Command: (leave empty — streams the full desktop)

---

## Step 6 – Install Moonlight on your devices

**iOS / iPad:** Search for **Moonlight** in the App Store — free.

**Android:** Available on Google Play and F-Droid.

**macOS:**
```bash
brew install --cask moonlight
```

---

## Step 7 – Connect Moonlight to Sunshine

1. Open Moonlight on your device
2. Moonlight will scan for Sunshine servers on your network — or add manually via Tailscale IP
3. Enter the PIN shown in Moonlight into the Sunshine web UI under **PIN**
4. Select an application and start streaming

---

## Streaming quality settings

In Sunshine web UI under **Configuration → Streaming**:

| Setting | Recommendation |
|---|---|
| Resolution | 1920x1080 or 2560x1440 |
| Frame rate | 60 FPS (or 120 FPS for supported displays) |
| Bitrate | 20-50 Mbps on local network, 10-20 Mbps over Tailscale |
| Encoder | NVENC H.265 (best quality/performance ratio) |
| AV1 | Enable if your client supports it — even better quality |

The RTX 4080 Super's NVENC encoder handles H.265 and AV1 with essentially zero performance impact on the game.

---

## Playing over Tailscale

Tailscale makes it possible to stream games from anywhere with acceptable latency on a good connection:

1. Connect your client device to Tailscale
2. In Moonlight, add the WCP machine by its Tailscale IP (`100.x.x.x`)
3. Connect and stream

Latency depends on your internet connection speed. On a home fibre connection (100+ Mbps upload), gaming over Tailscale from a nearby location works well.

---

## Useful commands

```bash
# Check Sunshine status
systemctl --user status sunshine

# Restart Sunshine
systemctl --user restart sunshine

# View Sunshine logs
journalctl --user -u sunshine -f
```

---

## What's next

Part 9 covers setting up the gaming library — Steam, Epic Games, GOG, and Ubisoft on Ubuntu with Proton for Windows game compatibility.

**Up next:** [Part 9 – Gaming Library on Ubuntu](../part-09-gaming-library/) *(coming soon)*

---

## Related guides

- [Tailscale – Getting Started](/guides/networking/tailscale-getting-started/) — required for remote streaming
- [Part 6 – Ollama and Open WebUI](../part-06-ollama/) — AI and gaming share the same GPU
- [Sunshine Documentation](https://docs.lizardbyte.dev/projects/sunshine/) — official docs
- [Moonlight Game Streaming](https://moonlight-stream.org) — official Moonlight site
