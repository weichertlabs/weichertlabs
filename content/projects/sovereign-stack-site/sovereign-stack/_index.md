---
title: Track B — Goose + Terraform + Ansible
description: Building the Sovereign Stack using an AI agent — what it managed autonomously, what broke, and what needed a human.
date: 2026-04-11
tags: [goose, terraform, ansible, ai-agent, deepseek, openrouter]
---

This track builds the entire Sovereign Stack using an AI agent. The agent is given a spec document, a set of tools, and access to the Proxmox API and SSH — then let loose.

The goal is not to show that AI can do everything perfectly. The goal is to find out exactly where the limits are, document them honestly, and build something useful along the way.

**Agent setup:**
- [Goose](https://github.com/block/goose) — open-source AI agent framework by Block
- DeepSeek V3 via [OpenRouter](https://openrouter.ai)
- Terraform (`bpg/proxmox` provider) for VM provisioning
- Ansible for in-VM configuration

{{< cards >}}
  {{< card link="part-01-ai-deployment" title="Part 1 – AI-Driven Deployment with Terraform and Goose" icon="terminal" >}}
{{< /cards >}}
