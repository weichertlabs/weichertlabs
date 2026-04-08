---
title: Getting Started with OpenCode – Terminal AI Coding Agent
description: How to install and configure OpenCode, the open-source terminal AI coding agent, on macOS with Ollama or other model providers.
date: 2026-04-05
tags: [ai, opencode, ollama, agent, local-ai, terminal, coding]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

OpenCode is an open-source AI coding agent that lives in your terminal. It brings LLM-powered assistance directly to your command line — reading files, editing code, running commands, and working through multi-step problems autonomously. It supports 75+ model providers including local models via Ollama, meaning you can run it entirely offline and privately.

## Goose vs OpenCode — What's the Difference?

If you've read the [Goose guide](/guides/ai/goose-getting-started), you might wonder why you'd use OpenCode instead. They're similar in many ways but have different strengths:

| | Goose | OpenCode |
|---|---|---|
| **Interface** | Desktop app + CLI | Terminal TUI (text UI) |
| **Focus** | General automation, MCP extensions, workflows | Code-focused, LSP integration |
| **Extensibility** | MCP extensions, Recipes | MCP, subagents, LSP |
| **Best for** | Controlling machines, general agent tasks | Coding, editing, refactoring |
| **Providers** | Any LLM | 75+ providers |
| **License** | Apache 2.0 | MIT |

The short version: if you want to automate tasks, control remote machines, or run general agentic workflows — Goose is the better fit. If you're working on code and want an agent that understands your project's language servers, imports, and diagnostics — OpenCode is purpose-built for that.

Both support local models via Ollama. Both are free and open source. Many people run both.

## What Makes OpenCode Stand Out

**LSP integration** — OpenCode automatically detects and connects to language servers for your project. This gives the model access to type information, compiler diagnostics, and code intelligence that most other tools lack. It understands your codebase the same way your editor does.

**Polished terminal UI** — OpenCode has a proper TUI with syntax highlighting, scrollable history, and a clean layout. It doesn't feel like an afterthought.

**Multi-session support** — you can run multiple agents in parallel on the same project, each handling different tasks without conflict.

**Plan and Build modes** — Plan mode lets the model reason and outline what it's about to do before touching any files. Build mode executes. This separation gives you control over when and how changes happen.

## Requirements

- macOS (Apple Silicon or Intel)
- Node.js 18+ (for npm install) or Homebrew
- An LLM provider — either:
  - [Ollama](https://ollama.com) running locally
  - An API key for OpenAI, Anthropic, Google Gemini, or similar

## Step 1 – Install OpenCode

**Via npm (recommended):**

```bash
npm install -g opencode-ai
```

**Via Homebrew:**

```bash
brew install opencode
```

**Via install script:**

```bash
curl -fsSL https://opencode.ai/install | bash
```

Verify it works:

```bash
opencode --version
```

## Step 2 – Configure a Model Provider

OpenCode stores its configuration in `~/.config/opencode/opencode.json`. Create or edit this file to set up your provider.

**Option A — Ollama (fully local)**

If Ollama is running on the same machine:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "ollama": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Ollama",
      "options": {
        "baseURL": "http://localhost:11434/v1"
      },
      "models": {
        "qwen2.5-coder:14b": {
          "tools": true
        }
      }
    }
  },
  "model": "ollama/qwen2.5-coder:14b"
}
```

If your Ollama instance is running on a different machine (e.g. a dedicated AI server), replace `localhost` with the IP:

```json
"baseURL": "http://10.10.1.190:11434/v1"
```

**Option B — OpenAI, Gemini, or other API**

Run `/connect` inside OpenCode and follow the prompts to authenticate with your provider of choice.

## Step 3 – Important: Context Window for Local Models

This is a common gotcha. Ollama defaults to a 4096 token context window for most models, even if the model supports much larger contexts. OpenCode needs at least 16k–32k tokens for agentic tool use to work reliably.

Create a custom model in Ollama with an increased context window:

```bash
ollama run qwen2.5-coder:14b
```

Inside the Ollama prompt:

```
/set parameter num_ctx 32768
/save qwen2.5-coder:14b-32k
/bye
```

Then update your OpenCode config to use the new model name:

```json
"models": {
  "qwen2.5-coder:14b-32k": {
    "tools": true
  }
}
```

Without this, tool calls may silently fail or produce garbage output as the model loses context mid-task.

## Step 4 – Run OpenCode

Navigate to a project directory and start OpenCode:

```bash
cd ~/your-project
opencode
```

On first run it will prompt you to select a provider and model. Once configured, you'll see the TUI interface. Type your instructions at the bottom prompt.

Some things to try:

```
Explain what this project does based on the file structure
```

```
Find all TODO comments in this codebase and summarize them
```

```
Refactor the authentication module to use async/await
```

## Step 5 – Plan vs Build Mode

By default OpenCode runs in **Build** mode, which means it will actually make file changes. Before running anything destructive, you can switch to **Plan** mode with `/plan` — the model will describe what it *would* do without touching any files. Review the plan, then switch back to build mode when you're happy.

This is particularly useful when you're working on unfamiliar code or giving OpenCode broad instructions.

## Model Recommendations for Local Use

As with Goose, model quality is the biggest variable. OpenCode is code-focused, so coding-specific models tend to perform better:

| Model | Size | Tool-calling | Notes |
|---|---|---|---|
| `qwen2.5-coder:14b` | ~9GB | Good | Best balance for 32GB machines |
| `qwen2.5-coder:32b` | ~19GB | Very good | Needs 32GB+ with headroom |
| `qwen3-coder:30b` | ~18GB | Excellent | Latest gen, strong tool use |
| `deepseek-coder-v2` | ~9GB | Good | Strong on code tasks |

Models under 14B tend to struggle with multi-step agentic tasks, especially when tool calls are involved. If OpenCode seems to be stalling or producing nonsensical tool calls, increasing the context window (Step 3) or switching to a larger model is the first thing to try.

## Connecting to a Remote Ollama Instance

If you have a more powerful machine running Ollama — say, a server with a discrete GPU — you can point OpenCode at it instead of running inference locally. Just update the `baseURL` in your config:

```json
"options": {
  "baseURL": "http://10.10.1.190:11434/v1"
}
```

No other changes needed. OpenCode doesn't care whether the model is local or remote, as long as the API is reachable.

## Related Links

- [OpenCode on GitHub](https://github.com/opencode-ai/opencode)
- [OpenCode documentation](https://opencode.ai/docs)
- [Ollama integration docs](https://docs.ollama.com/integrations/opencode)
- [Getting Started with Goose](/guides/ai/goose-getting-started)
- [Goose SSH Agent guide](/guides/ai/goose-ssh-agent)
