---
title: Controlling Remote Machines with Goose and SSH
description: How to use the Goose AI agent with the SSH MCP extension to autonomously manage and administer remote Linux machines using plain language.
date: 2026-04-05
tags: [ai, goose, ssh, ollama, agent, local-ai, linux]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

One of the more interesting things you can do with Goose is point it at a remote machine and let it work autonomously via SSH. Instead of logging in and running commands manually, you describe what you want in plain language and Goose handles the rest — checking disk usage, listing containers, restarting services, whatever you need.

This guide covers setting up the SSH extension for Goose and using it to manage remote Linux machines. It assumes you've already installed Goose and configured a model provider — if not, start with the [Getting Started guide](/guides/ai/goose-getting-started).

## How It Works

Goose uses the Model Context Protocol (MCP) to connect to external tools. There's an SSH MCP server called `ssh-mcp` that acts as a bridge: Goose instructs it to run a command, it connects to your remote machine via SSH, executes the command, and returns the output. Goose then reasons about the output and decides what to do next.

The whole flow stays local — your SSH key never leaves your machine, and no data goes to any cloud service (assuming you're using a local model or Claude Code).

## Requirements

- Goose installed and configured ([Getting Started guide](/guides/ai/goose-getting-started))
- Node.js installed (required for `npx`)
- SSH key-based authentication set up to the target machine
- The SSH private key must **not** have a passphrase (see below)

## Step 1 – Set Up SSH Key Authentication

If you don't already have key-based SSH access to your target machine, generate a dedicated key for Goose automation. It's good practice to use a separate key for automated access rather than your personal key.

```bash
# Generate a new key without a passphrase
# When asked for a passphrase, press Enter twice
ssh-keygen -t ed25519 -C "goose-automation" -f ~/.ssh/id_ed25519_goose
```

Copy the public key to your remote machine:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519_goose.pub youruser@10.10.1.163
```

Test that it works without a password prompt:

```bash
ssh -i ~/.ssh/id_ed25519_goose youruser@10.10.1.163
```

{{< callout type="warning" >}}
**Why no passphrase?** The `ssh-mcp` server reads the private key file directly and cannot interactively prompt for a passphrase. A key without a passphrase is standard practice for automation and CI/CD — just make sure the private key stays on your local machine and is never shared.
{{< /callout >}}

## Step 2 – Add the SSH Extension in Goose

Open Goose and go to **Extensions** → **Add custom extension**.

Fill in the following:

| Field | Value |
|---|---|
| Extension Name | `ssh-ubuntu` (or whatever the machine is) |
| Type | Standard IO (STDIO) |
| Description | Optional — e.g. "Ubuntu NUC at 10.10.1.163" |
| Command | See below |
| Timeout | 300 |

The command field should contain:

```
npx -y ssh-mcp -- --host=10.10.1.163 --port=22 --user=youruser --key=/Users/youruser/.ssh/id_ed25519_goose
```

Replace `10.10.1.163`, `youruser`, and the key path with your actual values.

Click **Add Extension**. Restart Goose for it to take effect.

## Step 3 – Test the Connection

Start a new chat. Be explicit about which extension to use, especially at first:

```
Use the ssh-ubuntu extension to run "uname -a" on the remote server
```

If it works, you'll see Goose call the SSH tool and return the output. From here you can give it more complex instructions:

```
Use the ssh-ubuntu extension to check disk usage and list any directories over 1GB
```

```
Use the ssh-ubuntu extension to show running Docker containers and their status
```

```
Use the ssh-ubuntu extension to check what's listening on port 80
```

## Gotchas From Real-World Testing

These are things that tripped up during actual setup — worth knowing before you run into them.

**The cask is called `block-goose`, not `goose`**

If you try `brew install --cask goose` you'll get an error. The correct command is:

```bash
brew install --cask block-goose
```

**Encrypted SSH keys don't work**

If your private key has a passphrase, you'll get this error:

```
MCP error -32603: Cannot parse privateKey: Encrypted private OpenSSH key detected, but no passphrase given
```

Generate a separate passphrase-free key specifically for Goose, as described in Step 1. ssh-agent doesn't help here — `ssh-mcp` reads the key file directly.

**Model choice matters enormously**

This is the biggest practical variable. With a weak model, Goose will see the SSH tool, form what looks like a correct tool call, but never actually execute it — instead looping through reasoning indefinitely, sometimes switching languages mid-conversation.

Models that work reliably for SSH tool-calling:

- **Claude** (via Claude Code CLI or API) — works first try, every time
- **qwen2.5:32b** — solid, requires ~30GB of unified memory
- **qwen2.5:14b** — inconsistent, sometimes works

Models that struggle:

- **gpt-oss:20b** and similar — generates tool call JSON but fails to execute it
- Anything under 14B — generally not reliable for agentic tasks

If Goose is reasoning endlessly without doing anything, switch models before debugging the extension.

**Be explicit about which extension to use**

Goose has multiple extensions active at once and will sometimes choose the wrong one. If you just say "check disk usage on 10.10.1.163", it might use the Developer extension and try to SSH locally instead of using the ssh-mcp extension. Being explicit — "use the ssh-ubuntu extension to..." — gets reliable results.

## Managing Multiple Machines

If you need to control several machines, add one extension per machine. Use descriptive names:

- `ssh-proxmox` → 10.10.1.10
- `ssh-ubuntu-nuc` → 10.10.1.163
- `ssh-kali` → 10.10.1.140

Each extension is configured independently with its own host and key. In a session, you can switch between them naturally:

```
Use ssh-proxmox to check VM status, then use ssh-ubuntu-nuc to verify the Docker stack is running
```

For managing many machines from a single extension, the `@fangjunjie/ssh-mcp-server` package supports a JSON config file with multiple hosts — worth looking at if you're managing a larger homelab.

## Example: Full Autonomous Check

Here's an example of what a capable model (Claude) can do in a single session with no manual intervention:

```
Use the ssh-ubuntu extension to:
1. Check disk usage on all mounted volumes
2. List running Docker containers and their uptime
3. Show the last 20 lines of the Docker daemon log
4. Summarize anything that looks unusual
```

Goose will run each command in sequence, parse the output, and give you a coherent summary — no manual SSH required.

## Related Links

- [Getting Started with Goose](/guides/ai/goose-getting-started)
- [ssh-mcp on GitHub](https://github.com/tufantunc/ssh-mcp)
- [Goose Extensions Directory](https://block.github.io/goose/v1/extensions/)
- [Ollama Model Library](https://ollama.com/library)
