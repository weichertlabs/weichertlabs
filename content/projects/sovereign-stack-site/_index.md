---
title: The Sovereign Stack
description: Building a fully open-source, AI-managed alternative to Microsoft 365 — provisioned and maintained by local AI agents on Proxmox.
date: 2026-04-11
tags: [proxmox, terraform, ansible, docker, ai, opnsense, wazuh, nextcloud]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

The Sovereign Stack is an experiment in replacing Microsoft 365 with a fully self-hosted, open-source infrastructure — and letting AI agents build and manage it.

Every service that a small business typically pays Microsoft for — email, file storage, office apps, identity management, video calls, monitoring — has a mature open-source alternative. The question this project tries to answer is whether AI agents can handle the deployment, configuration, and ongoing maintenance of that stack, and how honest we can be about where the limits are.

This is not a polished "just follow these steps" guide. It's a documented experiment. Things break. The agent gets confused. Permissions are wrong. The honest parts are as important as the parts that work.

---

## Two parallel tracks

Every part of this series is documented in two ways:

**Track A – Manual** walks through every step by hand. Every config file, every command, every mistake. Designed for understanding what you're building and why.

**Track B – Goose + Terraform + Ansible** builds the same infrastructure using an AI agent — [Goose](https://github.com/block/goose) with [DeepSeek V3](https://www.deepseek.com) via [OpenRouter](https://openrouter.ai) — combined with Terraform for VM provisioning and Ansible for in-VM configuration. The goal is to find out how much of a real infrastructure stack an AI agent can actually set up autonomously in 2026.

Both tracks build the same end result. Choose the one that fits your learning style — or follow both.

---

## The Stack

| Microsoft Product | Open Source Alternative |
|---|---|
| Azure AD / Entra ID | Authentik |
| Exchange / Outlook | Stalwart Mail |
| SharePoint / OneDrive | Nextcloud |
| Office 365 | Collabora Online (LibreOffice) |
| Teams | Nextcloud Talk |
| Azure Monitor / Defender | Wazuh + Grafana |
| Copilot | Ollama (local LLMs) |

---

## What you will build

| Phase | What gets built |
|---|---|
| **Phase 1** | Proxmox storage layout, OPNsense firewall, PBS backup server, VLAN networking |
| **Phase 2** | Core stack — Authentik, Nextcloud, Collabora Online, Stalwart Mail |
| **Phase 3** | Security — Wazuh SIEM, Suricata IDS, Prometheus, Grafana |
| **Phase 4** | AI agent integration — monitoring, alerting, auto-remediation |
| **Phase 5** | Evaluation — what the AI managed autonomously vs. what needed a human |

---

## Before you start

This project assumes you have a Proxmox VE host up and running. If you haven't done that yet:

- [Proxmox VE Post Install Script](https://weichertlabs.com/guides/virtualization/proxmox/proxmox-ve-post-install/) — clean up repos, apply tweaks, update the system

### Hardware used in this series

| Component | This series |
|---|---|
| **Host** | Gigabyte MC12-LE0 |
| **CPU** | AMD Ryzen 5800x (8 cores / 16 threads) |
| **RAM** | 128 GB |
| **Storage** | 120 GB NVMe (OS) + 1.9 TB NVMe (VMs) + 2× 3.6 TB HDD |
| **Network** | 10 GbE (Aquantia AQC107) |

### AI agent setup

- **Agent:** [Goose](https://github.com/block/goose) — open-source AI agent framework
- **Model:** DeepSeek V3 via [OpenRouter](https://openrouter.ai) (OpenAI-compatible API, cost-effective for code generation)
- **Alternatively:** [Claude Code](https://docs.anthropic.com/en/docs/claude-code) for more complex reasoning tasks
- **Provisioning:** Terraform with [bpg/proxmox](https://registry.terraform.io/providers/bpg/proxmox/latest) provider
- **Configuration:** Ansible over SSH

---

{{< cards >}}
  {{< card link="manual" title="Track A — Manual" subtitle="Step by step, every config, every mistake" icon="book-open" >}}
  {{< card link="ai-assisted" title="Track B — Goose + Terraform" subtitle="AI agent, Terraform, and Ansible automation" icon="terminal" >}}
{{< /cards >}}
