---
title: "Part 1 – Network Planning"
description: Designing the network architecture for the Proxmox security lab — VLAN segmentation, IP schema, and why each decision was made.
date: 2026-03-30
tags: [proxmox, networking, vlan, opnsense, security, homelab]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

Before a single VM is created, the network needs to be designed. A well-planned network makes everything else easier — firewall rules are cleaner, traffic flows are predictable, and isolating a compromised machine is straightforward.

This part covers the VLAN design, IP schema, and the reasoning behind each decision.

---

## Why VLAN segmentation?

Running everything on a flat network — where every VM can talk to every other VM — is fine for a basic lab but makes security work meaningless. If the attacker machine can reach the SIEM directly, the SIEM logs are the first thing that gets wiped.

VLAN segmentation solves this by putting different types of systems on different network segments, with a firewall controlling what can talk to what. This is how real networks are designed, and it's what makes the lab worth learning from.

In this lab, OPNsense acts as the software router and firewall between all VLANs. All inter-VLAN traffic passes through OPNsense, which means we have full visibility and control over every connection.

---

## VLAN Design

The 100-series was chosen deliberately — it avoids conflicts if the lab is later connected to a home network, which typically uses VLANs in the 1–99 range.

| VLAN | Name | Subnet | Purpose |
|---|---|---|---|
| **100** | Management | `10.10.100.0/24` | Proxmox web UI, OPNsense management, PBS |
| **110** | Servers | `10.10.110.0/24` | Wazuh SIEM, Windows domain controller, Linux servers |
| **120** | Clients | `10.10.120.0/24` | Windows 11 workstations, general client VMs |
| **130** | Security | `10.10.130.0/24` | Security tools, monitoring, threat intelligence |
| **140** | Pentest | `10.10.140.0/24` | Kali Linux — heavily restricted outbound |
| **150** | AI | `10.10.150.0/24` | Ollama, local AI models and tooling |
| **199** | Access | `10.10.199.0/24` | WireGuard VPN endpoint, remote access |

---

## Key isolation decisions

**Pentest VLAN (140) is the most restricted.**
Kali Linux lives here. Inbound access from other VLANs is blocked by default — you reach Kali from the Access VLAN via WireGuard. Outbound from Kali to other VLANs is controlled per exercise. Internet access from the Pentest VLAN is also restricted to prevent accidental data leakage during exercises.

**Management VLAN (100) is only reachable from Access VLAN (199).**
The Proxmox web interface and OPNsense management UI should never be reachable from the Servers or Pentest VLANs. If a server gets compromised, the management plane stays protected.

**AI VLAN (150) is isolated from Pentest VLAN (140) by default.**
In Phase 4 and 5, when local AI is added to both sides, controlled communication paths are opened explicitly. The default is no communication between these two segments.

**Servers VLAN (110) can send logs to Security VLAN (130).**
Wazuh agents on the servers need to reach the Wazuh manager. This is one of the few explicitly permitted cross-VLAN flows from the start.

---

## IP Schema

Each VLAN follows the same pattern:

| Address | Role |
|---|---|
| `.1` | OPNsense gateway |
| `.2` | Reserved |
| `.10–.49` | Static assignments (servers, infrastructure) |
| `.50–.99` | Reserved for future use |
| `.100–.254` | DHCP range |

**Static assignments used in this series:**

| IP | Hostname | Role |
|---|---|---|
| `10.10.100.1` | opnsense | OPNsense gateway (Management) |
| `10.10.100.10` | proxmox | Proxmox host management interface |
| `10.10.100.11` | pbs | Proxmox Backup Server |
| `10.10.110.1` | opnsense | OPNsense gateway (Servers) |
| `10.10.110.10` | wazuh | Wazuh SIEM manager |
| `10.10.110.11` | dc01 | Windows domain controller |
| `10.10.110.12` | linux01 | Linux server (web/app) |
| `10.10.110.13` | linux02 | Linux server (database) |
| `10.10.140.1` | opnsense | OPNsense gateway (Pentest) |
| `10.10.140.10` | kali | Kali Linux attack machine |
| `10.10.150.1` | opnsense | OPNsense gateway (AI) |
| `10.10.150.10` | ollama | Ollama AI server |
| `10.10.199.1` | opnsense | OPNsense gateway (Access) |

---

## Network diagram

```
                    ┌─────────────────────────────────┐
                    │         Proxmox VE Host          │
                    │                                  │
         ┌──────────┤         OPNsense VM              ├──────────┐
         │          │      (Router + Firewall)         │          │
         │          └──────────────────────────────────┘          │
         │                        │                               │
    WireGuard                     │                          Internet
    (VLAN 199)                    │
                     ┌────────────┼────────────┐
                     │            │            │
               VLAN 100     VLAN 110     VLAN 140
               Management   Servers      Pentest
               10.10.100    10.10.110    10.10.140
                  │            │            │
               Proxmox      Wazuh         Kali
               PBS          DC01
                            Linux VMs

               VLAN 120     VLAN 130     VLAN 150
               Clients      Security     AI
               10.10.120    10.10.130    10.10.150
                  │            │            │
               Win11        Tools        Ollama
```

---

## Remote access

Access to the lab from your own machine is via **WireGuard**, built into OPNsense. This gives you a secure tunnel into the Access VLAN (199), from where you can reach the management interfaces of all other VLANs — without exposing anything to the internet.

No port forwarding. No exposed web interfaces. Everything stays behind the WireGuard tunnel.

Alternatively, **Tailscale** can be installed on the Proxmox host itself for quick remote access during the build phase — see [Tailscale on Proxmox VE](/guides/networking/tailscale-proxmox/).

---

## What's next

With the network design in place, Part 2 sets up the OPNsense VM in Proxmox and configures the VLANs, DHCP, and firewall rules to match this design.

**Up next:** [Part 2 – OPNsense VM Setup](/projects/proxmox-home-lab/manual/part-02-opnsense/) *(coming soon)*

---

## Related guides

- [Proxmox VE Post Install Script](/guides/virtualization/proxmox/proxmox-ve-post-install/) — complete this before starting
- [Tailscale on Proxmox VE](/guides/networking/tailscale-proxmox/) — optional quick remote access during the build
- [Tailscale – Getting Started](/guides/networking/tailscale-getting-started/) — if you haven't set up Tailscale yet
