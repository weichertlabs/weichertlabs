---
title: Getting Started with Goose – Local AI Agent
description: How to install and configure Goose, the open-source AI agent by Block, on macOS with Ollama or Claude as the backend model provider.
date: 2026-04-05
tags: [ai, goose, ollama, agent, local-ai]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

Goose is an open-source AI agent developed by Block (the company behind Square and Cash App). Unlike a regular chat interface, Goose can actually *do things* — run shell commands, edit files, call external tools, and autonomously work through multi-step tasks. It connects to any LLM you choose, including local models via Ollama, which means your data can stay entirely on your own hardware.

This guide covers installing Goose on macOS and connecting it to a model provider. The follow-up guide covers using Goose to control remote machines via SSH.

## What Makes Goose Different

Most AI assistants suggest what you should do. Goose just does it. It uses the Model Context Protocol (MCP) to connect to tools and extensions, which means it can interact with your filesystem, run commands, call APIs, and more — all autonomously, based on plain language instructions.

Goose is available as both a desktop app and a CLI tool. They share the same configuration, so you can switch between them freely.

## Requirements

- macOS (Apple Silicon or Intel)
- Homebrew installed
- An LLM provider — either:
  - [Ollama](https://ollama.com) running locally (free, fully offline)
  - An Anthropic API key (for Claude)
  - Claude Code CLI installed (if you already have a Claude subscription)

## Step 1 – Install Goose

Goose is available via Homebrew as a cask. Note that the cask name is `block-goose`, not `goose` — there's a naming conflict with an unrelated database migration tool.

```bash
brew install --cask block-goose
```

Once installed, launch Goose from your Applications folder or Spotlight.

## Step 2 – Choose a Model Provider

On first launch, Goose will ask you to configure a provider. You have several options:

**Option A — Ollama (fully local, free)**

If you have Ollama running locally, select **Ollama** as your provider and set the API host to:

```
http://localhost:11434
```

Then choose a model. For Goose to work well with tools and extensions, model choice matters a lot — more on that below.

**Option B — Claude Code CLI**

If you already have Claude Code installed and a Claude subscription, select **Claude Code CLI** and enter `claude` as the command. Goose will use your existing Claude session. This is the easiest path to reliable tool-calling without API costs beyond your subscription.

**Option C — Anthropic API**

Select **Anthropic** and enter your API key. Choose `claude-haiku-3-5` for a cheap test, or `claude-sonnet-4-5` for better quality. Claude handles tool-calling extremely well.

## Step 3 – Configure Extensions

Extensions give Goose its superpowers. Go to **Extensions** in the sidebar. You'll see two sections: Default Extensions (already available) and Available Extensions (off by default).

**Recommended to enable:**

- **Developer** — runs shell commands, reads and writes files. Already on by default.
- **Computer Controller** — lets Goose control macOS via clicks and keyboard input.
- **Memory** — Goose learns your preferences over time.
- **Analyze** — useful for reading logs, config files, and code.

You can toggle extensions on/off per session as well, which is useful when you want to limit what Goose has access to.

## Step 4 – Start a Session

Click **Start New Chat** in the sidebar. Type instructions in plain language — Goose will figure out what tools to use.

Some examples to try:

```
Show me what's running on port 8080
```

```
Check disk usage on this machine and summarize it
```

```
List all running Docker containers
```

## A Note on Model Quality and Tool-Calling

This is important and often glossed over: **not all models handle tool-calling equally well.**

Goose works by asking the model to decide when and how to call tools. Smaller or less capable models often generate the correct tool call *syntax* in their response but fail to actually execute it — they reason endlessly in text instead of acting.

From hands-on testing:

| Model | Tool-calling reliability |
|---|---|
| Claude (any version) | Excellent — works first try |
| qwen2.5:32b | Good — reliable on most tasks |
| qwen2.5:14b | Inconsistent — works sometimes |
| gpt-oss:20b / Mistral variants | Poor — loops and hallucinates tool calls |
| Models under 14B | Generally not reliable for agentic use |

If Goose seems to be thinking endlessly without doing anything, the model is likely the bottleneck. Switch to a larger or more capable model.

**Hardware note:** Running a 32B model locally requires at least 32GB of unified memory (Apple Silicon) with plenty of headroom for the OS and other apps. On a Mac Studio M1 Max with 32GB, a 14B model is the practical sweet spot for reliable agentic use.

## Connecting to a Remote Ollama Instance

If you run Ollama on a more powerful machine (like a dedicated AI server with a discrete GPU), you can point Goose at it instead of running inference locally. In the provider settings, replace `localhost` with the IP of your Ollama server:

```
http://10.10.1.190:11434
```

This lets your Mac act as the agent frontend while a beefier machine handles inference — a useful setup if you have mixed hardware.

## Related Links

- [Goose on GitHub](https://github.com/block/goose)
- [Goose Extensions Directory](https://block.github.io/goose/v1/extensions/)
- [Ollama Model Library](https://ollama.com/library)
- [Using Goose to Control Remote Machines via SSH](/guides/ai/goose-ssh-agent)
