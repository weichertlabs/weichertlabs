---
title: Tailscale – Getting Started
description: How to install Tailscale and create a secure private network between all your devices — no port forwarding, no VPN server setup, just works.
date: 2026-03-27
tags: [networking, tailscale, vpn, remote-access, security]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

Tailscale is a zero-config VPN built on WireGuard. It connects all your devices — servers, laptops, phones — into a private network that works anywhere, without opening firewall ports or setting up a VPN server.

**Why Tailscale is great for home labs:**
- No port forwarding needed
- No VPN server to maintain
- Works behind NAT and firewalls automatically
- Each device gets a stable private IP (100.x.x.x)
- Free for personal use (up to 100 devices)
- Works on Linux, macOS, Windows, iOS, Android

---

## How Tailscale Works

Every device you add to Tailscale gets a private IP address in the `100.64.0.0/10` range. Devices can talk directly to each other using these IPs — encrypted, peer-to-peer, regardless of where they are physically located.

You manage everything from the Tailscale admin console at [login.tailscale.com](https://login.tailscale.com).

---

## Step 1 – Create a Tailscale Account

Go to [tailscale.com](https://tailscale.com) and sign up with your Google, GitHub, or Microsoft account. The free plan supports up to 100 devices — more than enough for a home lab.

---

## Step 2 – Install on Linux (Ubuntu/Debian)

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

Start Tailscale and authenticate:

```bash
sudo tailscale up
```

A URL will appear in the terminal. Open it in your browser and log in to authorize the device.

Check your Tailscale IP:

```bash
tailscale ip
```

---

## Step 3 – Install on macOS

```bash
brew install tailscale
```

Start and authenticate:

```bash
sudo tailscale up
```

Or install the macOS app from the App Store — it adds a menu bar icon for easy management.

---

## Step 4 – Connect Your Devices

Install Tailscale on all your devices — servers, laptops, phones. Each one authenticates with the same account and joins your private network automatically.

Once connected, every device can reach every other device using its Tailscale IP:

```bash
# SSH to your Proxmox server from anywhere
ssh root@100.x.x.x

# Or use the Tailscale hostname
ssh root@proxmox-node
```

---

## Useful Commands

| Command | What it does |
|---|---|
| `tailscale status` | Show all connected devices |
| `tailscale ip` | Show your Tailscale IP |
| `tailscale ping hostname` | Test connectivity to a device |
| `sudo tailscale up` | Connect to Tailscale |
| `sudo tailscale down` | Disconnect from Tailscale |
| `tailscale netcheck` | Check network connectivity |

---

## MagicDNS

Tailscale includes MagicDNS — automatic DNS for all your devices. Instead of remembering IP addresses, you can use hostnames:

```bash
ssh root@my-proxmox
ssh user@ubuntu-server
curl http://homeassistant:8123
```

Enable MagicDNS in the Tailscale admin console under **DNS → Enable MagicDNS**.

---

## Access Control

By default, all devices on your Tailscale network can reach each other. You can restrict access using ACLs (Access Control Lists) in the admin console — useful if you add other users to your network.

---

## Tips

- **Give devices descriptive names** in the admin console — makes MagicDNS much more useful
- **Use tags** to organize devices (e.g. `tag:servers`, `tag:lab`)
- **Key expiry** — by default, device keys expire after 180 days. Disable expiry for servers so they stay connected permanently

---

## Related Links

- [Tailscale on Proxmox VE](/guides/networking/tailscale-proxmox/) — access your entire Proxmox lab remotely
- [Tailscale Exit Node – Use Your Home Network as a VPN](/guides/networking/tailscale-exit-node/) — route all traffic through your home network
- [SSH Keys – The Right Way](/guides/operating-systems/linux/ssh-keys-linux/) — secure your SSH before exposing it over Tailscale
- [Tailscale Documentation](https://tailscale.com/kb/) — official docs
