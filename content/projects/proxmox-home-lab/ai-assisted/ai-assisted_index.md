---
title: Track B — Claude Code
description: Building the same Proxmox home lab using Claude Code, Terraform, and Ansible.
date: 2026-03-30
tags: [proxmox, claude-code, terraform, ansible, automation]
---

The same lab as Track A — but built using Claude Code with SSH access to the Proxmox server, Terraform for VM provisioning, and Ansible for configuration.

The question this track tries to answer: how much of a realistic security lab can an AI agent actually set up autonomously in 2026?

## The approach

Claude Code gets SSH access to the Proxmox server and a clear goal. It writes the Terraform configurations, runs them, writes the Ansible playbooks, and executes them. Where it gets stuck, that gets documented. Where it fails, that gets documented too.

No cherry-picking the successes.

## Status

| Part | Topic | Status |
|------|-------|--------|
| Part 1 | Network planning — letting Claude design the VLAN schema | Coming soon |
| Part 2 | OPNsense VM — Terraform + API automation | Coming soon |
| Part 3 | Wazuh SIEM — Ansible playbook deploy | Coming soon |
| Part 4 | Windows domain — WinRM + PowerShell remote | Coming soon |
| Part 5 | Linux servers — full Ansible provisioning | Coming soon |
| Part 6 | Integration — what Claude connected, what it couldn't | Coming soon |
| Bonus | Honest review — what worked, what didn't, what surprised us | Coming soon |

*Parts will be added as they are completed and tested.*
