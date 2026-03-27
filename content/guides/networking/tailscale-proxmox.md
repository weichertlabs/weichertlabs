---
title: Tailscale on Proxmox VE
description: How to install Tailscale on Proxmox VE to access your entire home lab remotely — SSH, web interface, and all VMs, from anywhere.
date: 2026-03-27
tags: [networking, tailscale, proxmox, remote-access, vpn]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

Installing Tailscale on your Proxmox host gives you secure remote access to your entire home lab from anywhere — the Proxmox web interface, all VMs, LXC containers, and services running on them. No port forwarding required.

Make sure you have Tailscale set up first: [Tailscale – Getting Started](/guides/networking/tailscale-getting-started/)

---

## Step 1 – Install Tailscale on Proxmox

SSH into your Proxmox host and run:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

---

## Step 2 – Start and Authenticate

```bash
sudo tailscale up
```

Open the URL shown in the terminal, log in to your Tailscale account, and authorize the device.

Check that it connected:

```bash
tailscale status
tailscale ip
```

Note the Tailscale IP — you'll use this to reach your Proxmox host remotely.

---

## Step 3 – Access Proxmox Web Interface Remotely

Once Tailscale is running on your Proxmox host, open the web interface from anywhere using the Tailscale IP:

```
https://100.x.x.x:8006
```

Or using MagicDNS (if enabled):

```
https://my-proxmox:8006
```

You'll get the full Proxmox web UI — create VMs, manage storage, check logs — all securely over Tailscale.

---

## Step 4 – SSH to Proxmox Remotely

```bash
ssh root@100.x.x.x

# Or with MagicDNS
ssh root@my-proxmox
```

---

## Step 5 – Access VMs and Containers

VMs and LXC containers running on Proxmox are reachable via the Proxmox host's Tailscale IP — but only if they're on the same network bridge.

**Option A – Install Tailscale on Each VM**

For full direct access, install Tailscale on each VM individually. Each VM gets its own Tailscale IP and can be reached directly:

```bash
# On the VM
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

**Option B – Use Proxmox as a Subnet Router**

Advertise your home lab subnet through Tailscale so all VMs are reachable without installing Tailscale on each one:

```bash
sudo tailscale up --advertise-routes=192.168.1.0/24
```

Then approve the subnet route in the Tailscale admin console under the device settings.

Now all devices on `192.168.1.0/24` — including all your VMs — are reachable from anywhere via Tailscale.

---

## Subnet Router vs Individual Install

| Approach | Pros | Cons |
|---|---|---|
| **Tailscale on each VM** | Direct peer-to-peer, most secure | More setup per VM |
| **Subnet router on Proxmox** | One install covers all VMs | Traffic routes through Proxmox host |

For a home lab, the subnet router approach is often the most practical — one install and your entire lab is accessible.

---

## Keep Tailscale Running After Reboot

Enable Tailscale to start automatically:

```bash
sudo systemctl enable tailscaled
```

Disable key expiry for your Proxmox nodes in the Tailscale admin console — servers should stay connected permanently without needing to re-authenticate.

---

## Useful Commands

| Command | What it does |
|---|---|
| `tailscale status` | Show all connected devices |
| `tailscale ip` | Show Tailscale IP of this node |
| `tailscale ping my-laptop` | Test connectivity to another device |
| `sudo tailscale up --advertise-routes=192.168.1.0/24` | Advertise local subnet |
| `sudo systemctl restart tailscaled` | Restart Tailscale |

---

## Related Links

- [Tailscale – Getting Started](/guides/networking/tailscale-getting-started/) — install Tailscale on all your devices
- [Tailscale Exit Node – Use Your Home Network as a VPN](/guides/networking/tailscale-exit-node/) — route traffic through your home network
- [Proxmox VE Post Install Script](/guides/virtualization/proxmox/proxmox-ve-post-install/) — set up Proxmox before adding Tailscale
- [Tailscale Subnet Router Documentation](https://tailscale.com/kb/1019/subnets/) — official docs
