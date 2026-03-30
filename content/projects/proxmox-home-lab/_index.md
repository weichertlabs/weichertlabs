---
title: Proxmox Home Lab – Security Lab Series
description: Build a complete security lab on Proxmox VE from scratch — network segmentation, SIEM, Windows domain, Linux servers, and local AI on both sides of the attack.
date: 2026-03-30
tags: [proxmox, opnsense, wazuh, security, homelab, kali, ai]
---

This project documents building a realistic, self-contained security lab on Proxmox VE — from a bare server to a full Red vs Blue scenario with local AI involved on both sides.

The lab covers network segmentation with OPNsense, centralised log collection and alerting with Wazuh, a Windows Active Directory domain, hardened Linux servers, and a dedicated Kali Linux attack machine. In the later phases, local AI (Ollama) is added to both the defensive and offensive sides.

The goal is a lab environment you can actually learn from — not a perfectly polished setup, but an honest one. Mistakes get documented. Dead ends get documented. What worked and what didn't, both get documented.

---

## Two parallel tracks

Every part of this series is documented in two ways:

**Track A – Manual** walks through every step by hand. Every config file, every command, every mistake. Designed for understanding what you're building and why.

**Track B – Claude Code** builds the same lab using Claude Code with SSH access to the Proxmox server, Terraform for VM provisioning, and Ansible for configuration. The goal is to find out how much a realistic security lab an AI agent can actually set up autonomously in 2026.

Both tracks build the same end result. Choose the one that fits your learning style — or follow both.

---

## What you will build

| Phase | What gets built |
|---|---|
| **Phase 1** | Network design, OPNsense router, VLAN segmentation |
| **Phase 2** | Wazuh SIEM, Windows domain, Linux servers |
| **Phase 3** | Kali Linux attack machine, pentest against the lab |
| **Phase 4** | Defensive local AI — Ollama + Wazuh integration |
| **Phase 5** | Offensive local AI — AI-assisted attacks from Kali |
| **Bonus** | Red vs Blue — full scenario, honest review of what AI contributed |

---

## Before you start

This project assumes you have a Proxmox VE host up and running with the basics sorted. If you haven't done that yet, complete these guides first:

- [Proxmox VE Post Install Script](/guides/virtualization/proxmox/proxmox-ve-post-install/) — clean up repos, apply tweaks, update the system
- [Run Proxmox Backup Server as a VM with Disk Passthrough](/guides/virtualization/proxmox/proxmox-backup-server-disk-passthrough/) — recommended before building anything you want to keep

### Minimum hardware requirements

This lab runs comfortably on a single Proxmox host. The setup used in this series:

| Component | This series | Minimum |
|---|---|---|
| **CPU** | Ryzen 5800x (8 cores / 16 threads) | 8 cores recommended |
| **RAM** | 128 GB | 32 GB minimum, 64 GB recommended |
| **Storage** | 2 TB NVMe | 500 GB+ |
| **Network** | 10 GbE | 1 GbE works fine |

The Windows domain and SIEM are the most RAM-hungry components. With 32 GB you can run the lab but will need to be selective about what runs simultaneously. With 64 GB+ everything runs comfortably at once.

### Software requirements

- **Proxmox VE 9.x** installed on bare metal
- Post-install script completed (repos cleaned up, system updated)
- Basic familiarity with the Proxmox web interface

No prior experience with OPNsense, Wazuh, or Active Directory is required — each component is built from scratch with full explanation.

---

{{< cards >}}
  {{< card link="manual" title="Track A — Manual" subtitle="Step by step, every config, every mistake" icon="book-open" >}}
  {{< card link="ai-assisted" title="Track B — Claude Code" subtitle="Terraform, Ansible, and AI automation" icon="sparkles" >}}
{{< /cards >}}
