---
title: "Part 2 – OPNsense VM & VLAN Configuration"
description: Setting up OPNsense in Proxmox with full VLAN segmentation, DHCP, and firewall rules for the security lab — following the network design from Part 1.
date: 2026-03-30
tags: [proxmox, opnsense, vlan, firewall, networking, homelab]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

This part sets up OPNsense as the central router and firewall for the security lab, implementing the VLAN design from [Part 1 – Network Planning](../part-01-network-planning/).

---

## Prerequisites

Before starting this part:

- ✅ OPNsense installed and accessible via web UI
  → Follow [OPNsense – Getting Started](/guides/networking/opnsense-getting-started/) first
- ✅ Network design reviewed
  → See [Part 1 – Network Planning](../part-01-network-planning/)
- ✅ Proxmox VE 9.x with post-install script completed

---

## The VLAN plan

As designed in Part 1, the lab uses seven VLANs:

| VLAN | Name | Subnet | Purpose |
|---|---|---|---|
| 100 | Management | `10.10.100.0/24` | Proxmox, OPNsense, PBS |
| 110 | Servers | `10.10.110.0/24` | Wazuh, Windows DC, Linux servers |
| 120 | Clients | `10.10.120.0/24` | Windows 11 workstations |
| 130 | Security | `10.10.130.0/24` | Security tooling |
| 140 | Pentest | `10.10.140.0/24` | Kali Linux — restricted |
| 150 | AI | `10.10.150.0/24` | Ollama and AI tooling |
| 199 | Access | `10.10.199.0/24` | WireGuard remote access |

---

## Step 1 – Set Up the Linux Bridge for VLANs in Proxmox

OPNsense needs a trunk interface — a single network interface that carries all VLANs tagged. In Proxmox, this is done with a Linux bridge with VLAN awareness enabled.

In Proxmox, go to **Node → Network → Edit** your internal bridge (e.g. `vmbr1`):

- Check **VLAN aware**
- Click **Apply**

This allows Proxmox to pass tagged VLAN traffic to VMs on this bridge.

---

## Step 2 – Add the Trunk Interface to OPNsense VM

The OPNsense VM needs a third network interface — the VLAN trunk:

1. Select the OPNsense VM → **Hardware → Add → Network Device**
2. Bridge: `vmbr1` (your VLAN-aware internal bridge)
3. **Leave VLAN tag empty** — this is a trunk, not an access port
4. Model: **VirtIO**
5. Click **Add**

This interface (`vtnet2` inside OPNsense) will carry all seven VLANs.

---

## Step 3 – Create VLAN Interfaces in OPNsense

In the OPNsense web UI:

Go to **Interfaces → Other Types → VLAN → Add**

Create one entry for each VLAN:

| Parent | VLAN tag | Description |
|---|---|---|
| `vtnet2` | 100 | VLAN100_Management |
| `vtnet2` | 110 | VLAN110_Servers |
| `vtnet2` | 120 | VLAN120_Clients |
| `vtnet2` | 130 | VLAN130_Security |
| `vtnet2` | 140 | VLAN140_Pentest |
| `vtnet2` | 150 | VLAN150_AI |
| `vtnet2` | 199 | VLAN199_Access |

Click **Save** after each entry.

---

## Step 4 – Assign VLAN Interfaces

Go to **Interfaces → Assignments**

Assign each VLAN to an interface slot:

| Interface | Network port | Rename to |
|---|---|---|
| OPT1 | VLAN100_Management | MGMT |
| OPT2 | VLAN110_Servers | SERVERS |
| OPT3 | VLAN120_Clients | CLIENTS |
| OPT4 | VLAN130_Security | SECURITY |
| OPT5 | VLAN140_Pentest | PENTEST |
| OPT6 | VLAN150_AI | AI |
| OPT7 | VLAN199_Access | ACCESS |

Click **Save**.

---

## Step 5 – Configure Each Interface

For each interface, go to **Interfaces → [Interface name]**:

- Enable the interface: ✅
- IPv4 configuration type: **Static IPv4**
- IPv4 address: the `.1` address for that VLAN

Example for MGMT:
- IP: `10.10.100.1`
- Subnet: `/24`

Repeat for all seven interfaces using the IP schema from Part 1.

Click **Save** and **Apply changes** after each interface.

---

## Step 6 – Configure DHCP for Each VLAN

Go to **Services → DHCPv4** and configure each VLAN:

| Interface | Range start | Range end |
|---|---|---|
| MGMT | `10.10.100.100` | `10.10.100.254` |
| SERVERS | `10.10.110.100` | `10.10.110.254` |
| CLIENTS | `10.10.120.100` | `10.10.120.254` |
| SECURITY | `10.10.130.100` | `10.10.130.254` |
| PENTEST | `10.10.140.100` | `10.10.140.254` |
| AI | `10.10.150.100` | `10.10.150.254` |
| ACCESS | `10.10.199.100` | `10.10.199.254` |

Enable DHCP on each interface and click **Save**.

---

## Step 7 – Firewall Rules

OPNsense blocks all inter-VLAN traffic by default. We need to explicitly allow the traffic flows that the lab requires.

### Default policy

The base policy for all VLANs — block everything, then allow specific flows.

Go to **Firewall → Rules** for each interface and ensure no allow-all rules exist.

### Allow DNS and DHCP (all VLANs)

On each VLAN interface, add a rule to allow DNS queries to OPNsense:

- Action: **Pass**
- Interface: the VLAN interface
- Protocol: **TCP/UDP**
- Destination: **This firewall**
- Destination port: **DNS (53)**
- Description: Allow DNS to OPNsense

### Management VLAN — allow access from Access VLAN only

On the MGMT interface:

- Allow: `10.10.199.0/24` → `10.10.100.0/24` (any port)
- Block: all other sources → `10.10.100.0/24`

### Servers VLAN — allow Wazuh agent traffic

On the SERVERS interface:

- Allow: `10.10.110.0/24` → `10.10.130.10` port **1514/1515** (Wazuh)

### Pentest VLAN — heavily restricted

On the PENTEST interface:

- Block: `10.10.140.0/24` → `10.10.100.0/24` (no management access)
- Block: `10.10.140.0/24` → `10.10.130.0/24` (no security access)
- Allow: `10.10.140.0/24` → `10.10.110.0/24` (target servers — controlled per exercise)
- Allow: `10.10.140.0/24` → WAN (internet — for tool updates, can be toggled)

### Access VLAN — WireGuard clients

On the ACCESS interface:

- Allow: `10.10.199.0/24` → any (WireGuard clients get full access — they are trusted)

{{< callout type="info" >}}
These are the baseline rules. As the lab evolves through each phase, additional rules will be added. Each part documents the specific rules needed for that phase.
{{< /callout >}}

---

## Step 8 – Set Up WireGuard for Remote Access

WireGuard is built into OPNsense and is the cleanest way to access the lab remotely.

Go to **VPN → WireGuard → Local**:

1. Click **Add**
2. **Name:** `wg-lab`
3. **Listen port:** `51820`
4. Click **Generate** to create the server key pair
5. **Tunnel address:** `10.10.199.1/24`
6. Click **Save**

### Add a peer (your machine)

On your local machine, install WireGuard and generate a key pair:

```bash
# macOS
brew install wireguard-tools
wg genkey | tee privatekey | wg pubkey > publickey
cat publickey
```

Back in OPNsense → **VPN → WireGuard → Peers → Add:**

- **Name:** your machine name
- **Public key:** paste your public key
- **Allowed IPs:** `10.10.199.2/32`

### WireGuard client config (on your machine)

```ini
[Interface]
PrivateKey = YOUR_PRIVATE_KEY
Address = 10.10.199.2/32
DNS = 10.10.100.1

[Peer]
PublicKey = OPNSENSE_PUBLIC_KEY
Endpoint = YOUR_PUBLIC_IP:51820
AllowedIPs = 10.10.0.0/16
PersistentKeepalive = 25
```

Enable WireGuard in OPNsense: **VPN → WireGuard → General → Enable WireGuard**

Add a firewall rule on the WAN interface to allow UDP port 51820.

---

## Verification

Test that the VLANs are working correctly:

```bash
# From a VM on VLAN 110, ping the gateway
ping 10.10.110.1

# From your machine via WireGuard, ping Proxmox
ping 10.10.100.10

# Verify blocked traffic — from a VLAN 140 VM, try to reach management
ping 10.10.100.1  # should fail
```

---

## What's next

With OPNsense configured and all VLANs in place, Part 3 deploys the Wazuh SIEM on the Servers VLAN and begins enrolling agents.

**Up next:** Part 3 – Wazuh SIEM Deployment *(coming soon)*

---

## Related guides

- [OPNsense – Getting Started](/guides/networking/opnsense-getting-started/) — install OPNsense from scratch
- [Part 1 – Network Planning](../part-01-network-planning/) — the VLAN design this part implements
- [Tailscale on Proxmox VE](/guides/networking/tailscale-proxmox/) — alternative remote access during the build phase
