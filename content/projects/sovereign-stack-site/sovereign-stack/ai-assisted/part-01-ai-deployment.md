---
title: "Part 1 – AI-Driven Deployment with Terraform and Goose"
description: Using an AI agent to provision the first VM on Proxmox — what worked, what broke, and what the agent figured out on its own.
date: 2026-04-11
tags: [proxmox, terraform, goose, ai-agent, opnsense, deepseek, openrouter]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

This is where the experiment begins. Instead of manually writing Terraform code and provisioning VMs the traditional way, the goal is to hand a spec document to an AI agent and see how far it gets on its own.

The short version: it got further than expected, broke in interesting ways, and the failure modes were just as useful as the successes.

## Tools Used

- **[Goose](https://github.com/block/goose)** — open-source AI agent framework by Block. Runs locally, supports MCP extensions, and can execute shell commands, write files, and run arbitrary tools.
- **[DeepSeek V3](https://www.deepseek.com)** via **[OpenRouter](https://openrouter.ai)** — the model driving the agent. OpenRouter gives you access to a wide range of models through a single OpenAI-compatible API endpoint. Cost-effective and solid for code generation and Terraform/Ansible tasks.
- **[Claude Code](https://docs.anthropic.com/en/docs/claude-code)** — used for more complex reasoning tasks and reviewing generated infrastructure code.
- **[Terraform](https://www.terraform.io)** with the **[bpg/proxmox provider](https://registry.terraform.io/providers/bpg/proxmox/latest)** — handles VM provisioning via the Proxmox API.

## Prerequisites

Before the agent can do anything, a few things need to be in place manually:

**On Proxmox:**
```bash
# Create a dedicated API user for the agent
pveum user add goose@pve --comment "Goose automation"
pveum aclmod / --user goose@pve --role Administrator
pveum user token add goose@pve automation --privsep 0
```

Start with the `Administrator` role for initial setup. Scope it down once everything is working — this is a lab, and debugging permission issues while also debugging agent behaviour is frustrating.

**On your workstation (Mac Studio in this case):**
```bash
# Install Terraform
brew install terraform

# Create project structure
mkdir -p ~/sovereign-stack/{terraform,ansible,docker,goose,docs}

# Create SSH key for agent access to VMs
ssh-keygen -t ed25519 -C "sovereign-stack-agent" -f ~/.ssh/id_ed25519_goose
ssh-copy-id -i ~/.ssh/id_ed25519_goose.pub root@<proxmox-ip>
```

**Goose setup:**

Install Goose desktop or CLI from [github.com/block/goose](https://github.com/block/goose). Configure it with DeepSeek via OpenRouter:

- Provider: OpenAI-compatible
- Host: `https://openrouter.ai/api/v1`
- Model: `deepseek/deepseek-chat-v3-5`
- API key: your OpenRouter key

Enable the **Developer** extension in Goose — this gives it shell access to run Terraform, Ansible, and arbitrary commands locally.

## The Terraform Provider

One non-obvious requirement: the `bpg/proxmox` provider needs **both** API access and SSH access to the Proxmox host. API alone isn't enough — it uses SSH for disk operations. Your `versions.tf` needs an SSH block:

```hcl
terraform {
  required_providers {
    proxmox = {
      source  = "bpg/proxmox"
      version = "~> 0.73"
    }
  }
}

provider "proxmox" {
  endpoint  = "https://<proxmox-ip>:8006/"
  api_token = "goose@pve!automation=<your-token>"
  insecure  = true

  ssh {
    agent       = false
    username    = "root"
    private_key = file("~/.ssh/id_ed25519_goose")
  }
}
```

Without the `ssh {}` block, you'll get this error when applying:

```
Error: creating custom disk: unable to authenticate user "" over SSH
```

## Verifying the Connection

Before unleashing the agent, verify that Terraform can reach Proxmox:

```hcl
# main.tf — simple node listing
data "proxmox_virtual_environment_nodes" "available_nodes" {}

output "nodes" {
  value = data.proxmox_virtual_environment_nodes.available_nodes.names
}
```

```bash
cd terraform
terraform init
terraform plan
# Expected output: nodes = ["pvegb"]
```

If you see your node name, the connection works. Then hand it to the agent.

## The Agent Prompt

The agent was given a spec document describing the full project and a focused first task:

```
You are deploying the Sovereign Stack – a fully open-source alternative 
to Microsoft 365, managed by AI agents on Proxmox.

Read the full project spec at:
/path/to/sovereign-stack/docs/spec.md

Your first task is Phase 1, Step 1:
Write Terraform code to provision the OPNsense VM on Proxmox:
- VM ID: 100, Name: opnsense
- RAM: 2GB, Disk: 20GB on intelpool
- ISO: OPNsense-25.1-dvd-amd64.iso (already on host)
- Network: vmbr0 bridge

Do NOT apply yet — show me terraform plan output first and 
wait for my approval before running terraform apply.

Constraints:
- Never touch rpool
- Never delete ISOs
- Always ask before applying
```

## What the Agent Did

The agent worked through the task methodically — reading the spec, inspecting existing files, generating Terraform code, running `terraform plan`, and iterating on errors. Here's a condensed version of what happened:

**Round 1 — First attempt at terraform plan**

The agent generated a working VM resource but with a few issues it caught itself on the next run:
- Used `enabled` attribute in the `cdrom` block (deprecated in newer provider versions)
- Set `operating_system type = "l26"` (Linux) — wrong for OPNsense which is FreeBSD-based
- Added a `boot_order` referencing `scsi0` when the disk interface was `virtio0`

**Round 2 — After self-correction**

The agent fixed all three issues without being told. It produced a clean plan. At this point human review flagged two more things:

- The `initialization {}` block (cloud-init) doesn't work on OPNsense/FreeBSD — remove it
- The `agent {}` block (QEMU Guest Agent) isn't supported on FreeBSD — remove it

**Round 3 — Permission errors**

`terraform apply` failed with storage permission errors. The agent correctly identified that the API token needed storage permissions — but then incorrectly assumed the ISO storage was named `local`. On this host, it's named `isos`.

Checking actual storage names on Proxmox:
```bash
pvesm status
# Shows: intelpool (zfspool), isos (dir), local (disabled)
```

The fix — update the ISO path variable:
```
opnsense_iso_path = "isos:iso/OPNsense-25.1-dvd-amd64.iso"
```

And add the missing storage permission:
```bash
pveum aclmod /storage/isos --user goose@pve --role PVEDatastoreUser
```

**Round 4 — Disk format error**

After fixing storage names, a new error appeared:

```
Error: creating custom disk: unable to parse volume ID 'intelpool:'
```

The agent had set `file_id = "intelpool:vm-100-disk-0"` in the disk block. For new disks, `file_id` should be omitted entirely — Proxmox creates it automatically. The correct disk block:

```hcl
disk {
  datastore_id = var.storage_pool
  size         = 20
  interface    = "virtio0"
  iothread     = true
  discard      = "on"
}
```

**Final apply — success:**

```
proxmox_virtual_environment_vm.opnsense: Creation complete after 3s [id=100]
Apply complete! Resources: 1 added, 0 changed, 1 destroyed.
```

## Where the Agent Needed Human Input

| Issue | Agent's Action | Human Input Required |
|---|---|---|
| Deprecated `cdrom` attribute | Fixed autonomously | None |
| Wrong OS type (l26 vs other) | Fixed after being told | One correction |
| cloud-init block on FreeBSD | Fixed after being told | One correction |
| Wrong storage name (local vs isos) | Couldn't know — not in spec | Check `pvesm status`, update spec |
| Missing SSH block in provider | Couldn't know — provider quirk | Add `ssh {}` block to versions.tf |
| `file_id` for new disks | Fixed after being told | One correction |

## The Loop Problem

One failure worth documenting: when the agent hit the `Sys.Modify` permission error, it ran `terraform apply` repeatedly without getting new information. It also started considering switching from Terraform to bash scripts on the Proxmox host — a reasonable workaround, but not what we wanted.

**Lesson:** AI agents need explicit stopping conditions. A prompt like "if you hit the same error twice, stop and explain what you need rather than retrying" would have prevented this. This goes into the spec for future sessions.

## OPNsense Initial Setup

After `terraform apply`, the VM boots from the OPNsense ISO automatically. Connect to the console via Proxmox web UI and complete the installation:

1. Select **Install (UFS)** — no need for ZFS in a VM
2. Select the `vtbd0` disk (the 20GB virtio disk created by Terraform)
3. Set a root password
4. Reboot — then remove the ISO from the VM's CD-ROM drive before it boots again

After boot, set the management IP via the console menu:
- Option `2` — Set interface IP address
- LAN interface → static IP → `10.10.0.1/24`
- No DHCP, no upstream gateway

OPNsense web UI is then available at `https://10.10.0.1` from a client on VLAN 100.

## What's Next

Part 2 covers provisioning the Proxmox Backup Server (PBS) VM and setting up the storage pools — again driven by the agent, with the lessons from Part 1 baked into the spec and constraints.

## Related Links

- [Goose — open-source AI agent framework](https://github.com/block/goose)
- [OpenRouter — multi-model API gateway](https://openrouter.ai)
- [bpg/proxmox Terraform provider](https://registry.terraform.io/providers/bpg/proxmox/latest)
- [OPNsense](https://opnsense.org)
