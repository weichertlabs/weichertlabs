---
title: Oh My Zsh – Supercharge Your Terminal on macOS
description: How to install and configure Oh My Zsh on macOS — themes, plugins, and aliases that make your terminal faster and more enjoyable to use.
date: 2026-03-27
tags: [macos, terminal, zsh, oh-my-zsh, productivity]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

Oh My Zsh is an open-source framework for managing your zsh configuration. It comes with hundreds of plugins, themes, and helpers that make your terminal significantly more powerful and enjoyable to use.

**What you get:**
- A much nicer looking prompt with useful info
- Autocompletion for git, docker, kubectl, and more
- Syntax highlighting and command suggestions
- Easy plugin and theme management

## Requirements

- macOS with zsh (default since Catalina)
- [Homebrew](/guides/operating-systems/macos/install-homebrew-macos/) installed
- Git installed (`brew install git`)

---

## Step 1 – Install Oh My Zsh

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

The installer will:
- Back up your existing `~/.zshrc` to `~/.zshrc.pre-oh-my-zsh`
- Create a new `~/.zshrc` with Oh My Zsh configuration
- Set Oh My Zsh as the default zsh framework

Restart your terminal or run:

```bash
source ~/.zshrc
```

---

## Step 2 – Change the Theme

Oh My Zsh comes with many built-in themes. The default is `robbyrussell`.

Open your config:

```bash
nano ~/.zshrc
```

Find this line and change the theme:

```bash
ZSH_THEME="robbyrussell"
```

**Popular built-in themes:**

| Theme | Description |
|---|---|
| `robbyrussell` | Default — clean and minimal |
| `agnoster` | Shows git branch and status (requires Powerline font) |
| `af-magic` | Colorful, shows user and path clearly |
| `bira` | Multi-line prompt with git info |
| `simple` | Very minimal |

To see all available themes:

```bash
ls ~/.oh-my-zsh/themes/
```

Or browse them at [github.com/ohmyzsh/ohmyzsh/wiki/Themes](https://github.com/ohmyzsh/ohmyzsh/wiki/Themes).

Save and reload:

```bash
source ~/.zshrc
```

---

## Step 3 – Install a Powerline Font (Optional)

Some themes like `agnoster` require a Nerd Font to display icons correctly.

Install via Homebrew:

```bash
brew tap homebrew/cask-fonts
brew install --cask font-meslo-lg-nerd-font
```

Then set the font in your terminal:
- **Terminal.app:** Preferences → Profiles → Text → Change Font
- **iTerm2:** Preferences → Profiles → Text → Font

---

## Step 4 – Enable Plugins

Plugins add autocompletion and shortcuts for specific tools. Edit `~/.zshrc`:

```bash
nano ~/.zshrc
```

Find the plugins line and add what you need:

```bash
plugins=(
  git
  docker
  docker-compose
  brew
  macos
  zsh-autosuggestions
  zsh-syntax-highlighting
)
```

**Built-in plugins (no installation needed):**

| Plugin | What it adds |
|---|---|
| `git` | Git aliases and autocompletion |
| `docker` | Docker command autocompletion |
| `docker-compose` | Compose autocompletion |
| `brew` | Homebrew aliases |
| `macos` | macOS-specific commands |
| `sudo` | Press ESC twice to add sudo to last command |
| `copypath` | Copy current path to clipboard |

---

## Step 5 – Install Popular Third-Party Plugins

Two of the most useful plugins require separate installation:

**zsh-autosuggestions** – suggests commands as you type based on history:

```bash
git clone https://github.com/zsh-users/zsh-autosuggestions \
  ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

**zsh-syntax-highlighting** – highlights commands in green (valid) or red (invalid):

```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting \
  ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

Add both to your plugins list in `~/.zshrc`, then reload:

```bash
source ~/.zshrc
```

---

## Step 6 – Install Powerlevel10k (Optional but Recommended)

Powerlevel10k is the most popular Oh My Zsh theme — fast, highly configurable, and looks great.

```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git \
  ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```

Set it as your theme in `~/.zshrc`:

```bash
ZSH_THEME="powerlevel10k/powerlevel10k"
```

Reload and follow the interactive configuration wizard:

```bash
source ~/.zshrc
```

The wizard walks you through choosing your preferred style — icons, colors, prompt elements — and saves everything automatically.

---

## Useful Oh My Zsh Commands

| Command | What it does |
|---|---|
| `omz update` | Update Oh My Zsh |
| `omz reload` | Reload configuration |
| `omz plugin list` | List available plugins |
| `omz theme list` | List available themes |
| `omz theme set robbyrussell` | Switch theme without editing config |

---

## Keep Your Aliases

Oh My Zsh creates a new `~/.zshrc` but your aliases still belong there. Add them at the bottom of `~/.zshrc`, or keep them in a separate `~/.zsh_aliases` file and source it:

```bash
# Add to the bottom of ~/.zshrc
source ~/.zsh_aliases
```

---

## Related Links

- [Oh My Zsh Official Site](https://ohmyz.sh) — documentation and plugin list
- [Powerlevel10k Theme](https://github.com/romkatv/powerlevel10k) — the most popular Oh My Zsh theme
- [Bash/Zsh Aliases on macOS](/guides/operating-systems/macos/aliases-macos/) — combine aliases with Oh My Zsh for maximum productivity
- [Essential macOS Terminal Commands](/guides/operating-systems/macos/essential-macos-terminal-commands/) — macOS terminal basics
