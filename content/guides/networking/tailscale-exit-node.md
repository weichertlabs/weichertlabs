---
title: Tailscale Exit Node – Use Your Home Network as a VPN
description: How to set up a Tailscale exit node on Linux to route all your internet traffic through your home network when you're away.
date: 2026-03-27
tags: [networking, tailscale, vpn, exit-node, privacy]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

A Tailscale exit node lets you route all your internet traffic through a specific device — like a server at home. When you're on a public WiFi at a café or hotel, all your traffic exits through your home network instead. Think of it as a personal VPN using your own hardware.

Make sure you have Tailscale set up first: [Tailscale – Getting Started](/guides/networking/tailscale-getting-started/)

---

## What is an Exit Node?

Normally, Tailscale only routes traffic between your devices — your laptop talks to your home server, but general internet traffic goes straight out wherever you are.

With an exit node, you tell Tailscale to route **all** internet traffic through a specific device. That device becomes your gateway to the internet — your home IP address is used for all connections.

**Use cases:**
- Secure browsing on public WiFi
- Access services that only allow your home IP
- Privacy when traveling
- Access geo-restricted content available at home

---

## Step 1 – Set Up the Exit Node (on Your Linux Server)

Choose a device to be your exit node — ideally a server that's always on, like a Linux VM or your Proxmox host.

Enable IP forwarding (required for exit node functionality):

```bash
echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf
echo 'net.ipv6.conf.all.forwarding = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf
sudo sysctl -p /etc/sysctl.d/99-tailscale.conf
```

Advertise this device as an exit node:

```bash
sudo tailscale up --advertise-exit-node
```

---

## Step 2 – Approve the Exit Node

In the Tailscale admin console at [login.tailscale.com](https://login.tailscale.com):

1. Go to **Machines**
2. Find your exit node device
3. Click the three dots menu → **Edit route settings**
4. Enable **"Use as exit node"**
5. Click **Save**

---

## Step 3 – Use the Exit Node on Your Other Devices

**On Linux:**

```bash
# Use a specific exit node
sudo tailscale up --exit-node=my-home-server

# Or use the Tailscale IP
sudo tailscale up --exit-node=100.x.x.x

# Disable exit node (go back to direct internet)
sudo tailscale up --exit-node=
```

**On macOS:**
1. Click the Tailscale menu bar icon
2. Go to **Exit Node**
3. Select your home server

**On iOS/Android:**
1. Open the Tailscale app
2. Tap **Exit Node**
3. Select your home server

---

## Verify It's Working

Check your public IP — it should show your home IP address:

```bash
curl ifconfig.me
```

Or visit [whatismyip.com](https://whatismyip.com) in your browser — you should see your home network's public IP.

---

## Exit Node + Subnet Router Together

You can combine exit node and subnet router on the same device — giving you both full internet routing through home AND access to all local devices:

```bash
sudo tailscale up \
  --advertise-exit-node \
  --advertise-routes=192.168.1.0/24
```

---

## Allow Local Network Access While Using Exit Node

By default, when using an exit node, you can't access devices on the local network where you physically are (e.g. a printer at a hotel). Enable local access:

```bash
sudo tailscale up \
  --exit-node=my-home-server \
  --exit-node-allow-lan-access
```

---

## Tips

- **Always-on server** — your exit node needs to be always on. A Linux VM or a small always-on machine works best.
- **Bandwidth** — all your internet traffic goes through your home connection. Make sure your home upload speed is sufficient.
- **Multiple exit nodes** — you can have multiple exit nodes in different locations and switch between them.

---

## Related Links

- [Tailscale – Getting Started](/guides/networking/tailscale-getting-started/) — set up Tailscale first
- [Tailscale on Proxmox VE](/guides/networking/tailscale-proxmox/) — run the exit node on Proxmox
- [Tailscale Exit Node Documentation](https://tailscale.com/kb/1103/exit-nodes/) — official docs
