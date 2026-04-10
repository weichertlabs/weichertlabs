---
title: Windows Terminal – Setup and Tips
description: How to install and configure Windows Terminal on Windows 11 — profiles for PowerShell, CMD, and WSL, themes, keyboard shortcuts, and tips for a better terminal experience.
date: 2026-04-10
tags: [windows, terminal, powershell, wsl, productivity, setup]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

Windows Terminal is Microsoft's modern terminal application — tabbed, customizable, and able to run PowerShell, CMD, WSL (Linux), and SSH sessions side by side. It replaces the old individual windows for each shell and is the best terminal experience available on Windows.

---

## Installation

Windows Terminal comes pre-installed on Windows 11. On Windows 10, install it:

```powershell
winget install Microsoft.WindowsTerminal
```

Or from the **Microsoft Store** — search for "Windows Terminal".

---

## First launch

Open Windows Terminal from:
- **Start menu:** Search "Terminal"
- **Right-click Desktop** → "Open in Terminal"
- **Right-click a folder** → "Open in Terminal" (opens in that folder)
- **Keyboard:** `Win+X` → "Terminal" or "Terminal (Admin)"

The default profile is PowerShell. The dropdown arrow next to the `+` tab shows all available profiles.

---

## Key keyboard shortcuts

| Shortcut | Action |
|---|---|
| `CTRL+T` | New tab (default profile) |
| `CTRL+SHIFT+T` | New tab (same profile) |
| `CTRL+W` | Close tab |
| `CTRL+TAB` | Next tab |
| `CTRL+SHIFT+TAB` | Previous tab |
| `ALT+SHIFT+D` | Split pane (horizontal) |
| `ALT+SHIFT+MINUS` | Split pane (vertical) |
| `ALT+Arrow` | Navigate between panes |
| `CTRL+SHIFT+P` | Command palette |
| `CTRL+,` | Open Settings |
| `CTRL+SHIFT+F` | Search in terminal |
| `CTRL+SHIFT+1` | Open profile 1 |
| `CTRL+SHIFT+2` | Open profile 2 |
| `F11` | Fullscreen |

---

## Settings overview

Open Settings with `CTRL+,` or through the dropdown menu. Settings has two modes:

- **GUI settings** — click-based configuration
- **settings.json** — full JSON configuration (click "Open JSON file" in the bottom left)

Most customization is easiest done in the JSON file.

---

## Profiles

Each profile in Windows Terminal is a separate shell configuration. Common profiles:

- **Windows PowerShell** — built-in PowerShell 5.1
- **PowerShell** — PowerShell 7+ (if installed)
- **Command Prompt** — classic CMD
- **Ubuntu** or other WSL distributions (added automatically when WSL is installed)

### Setting a default profile

In Settings → Startup → Default profile, select PowerShell 7 if installed:

```json
"defaultProfile": "{your-powershell-7-guid}"
```

Or in the GUI: **Settings → Startup → Default profile** → select from dropdown.

---

## Customizing profiles

In `settings.json`, each profile looks like:

```json
{
    "profiles": {
        "defaults": {
            "font": {
                "face": "CaskaydiaCove Nerd Font",
                "size": 12
            },
            "opacity": 95,
            "useAcrylic": true
        },
        "list": [
            {
                "name": "PowerShell",
                "source": "Windows.Terminal.PowershellCore",
                "startingDirectory": "%USERPROFILE%",
                "icon": "ms-appx:///ProfileIcons/pwsh.png"
            },
            {
                "name": "Ubuntu",
                "source": "Windows.Terminal.Wsl",
                "startingDirectory": "//wsl$/Ubuntu/home/patrik",
                "colorScheme": "One Half Dark"
            }
        ]
    }
}
```

---

## Fonts — install a Nerd Font

For best results with Oh My Posh or other prompt themes, install a Nerd Font:

1. Download **CaskaydiaCove Nerd Font** from [nerdfonts.com](https://www.nerdfonts.com/font-downloads)
2. Extract and install the `.ttf` files (right-click → Install for all users)
3. Set in Windows Terminal: **Settings → Profiles → Appearance → Font face**

Or via PowerShell:

```powershell
# Install Oh My Posh (includes font management)
winget install JanDeDobbeleer.OhMyPosh

# Install CaskaydiaCove Nerd Font
oh-my-posh font install CaskaydiaCove
```

---

## Color schemes

Windows Terminal includes built-in color schemes. Change them in **Settings → Profiles → Appearance → Color scheme**.

Popular built-in schemes:
- **One Half Dark** — clean dark theme
- **Campbell** — classic Windows Terminal default
- **Tango Dark** — similar to classic Linux terminals
- **Solarized Dark** — popular developer theme

Add a custom scheme in `settings.json`:

```json
"schemes": [
    {
        "name": "MyScheme",
        "background": "#1E1E2E",
        "foreground": "#CDD6F4",
        "black": "#45475A",
        "red": "#F38BA8",
        "green": "#A6E3A1",
        "yellow": "#F9E2AF",
        "blue": "#89B4FA",
        "purple": "#CBA6F7",
        "cyan": "#94E2D5",
        "white": "#BAC2DE"
    }
]
```

---

## Oh My Posh — a better prompt

Oh My Posh adds a rich, informative prompt to PowerShell — showing git branch, exit codes, execution time, and more.

```powershell
# Install
winget install JanDeDobbeleer.OhMyPosh

# Add to your PowerShell profile
notepad $PROFILE
```

Add this line:

```powershell
oh-my-posh init pwsh --config "$env:POSH_THEMES_PATH\jandedobbeleer.omp.json" | Invoke-Expression
```

Browse themes:

```powershell
Get-PoshThemes
```

Pick one and update the config path:

```powershell
oh-my-posh init pwsh --config "$env:POSH_THEMES_PATH\catppuccin_mocha.omp.json" | Invoke-Expression
```

---

## Useful settings.json tweaks

```json
{
    "initialCols": 120,
    "initialRows": 30,
    "copyOnSelect": true,
    "profiles": {
        "defaults": {
            "bellStyle": "none",
            "scrollbarState": "hidden",
            "historySize": 9001
        }
    },
    "actions": [
        {
            "command": "unbound",
            "keys": "ctrl+shift+w"
        },
        {
            "command": {
                "action": "splitPane",
                "split": "auto"
            },
            "keys": "alt+shift+d"
        }
    ]
}
```

---

## Running as Administrator

Some commands require admin privileges. Options:

1. **Right-click** Windows Terminal in Start → "Run as administrator"
2. **CTRL+SHIFT+click** on a profile in the tab dropdown
3. Add an admin profile to settings:

```json
{
    "name": "PowerShell (Admin)",
    "commandline": "pwsh.exe -NoExit -Command Start-Process pwsh -Verb RunAs",
    "icon": "ms-appx:///ProfileIcons/pwsh.png"
}
```

---

## SSH sessions as profiles

Add SSH connections as named profiles in Windows Terminal:

```json
{
    "name": "Proxmox",
    "commandline": "ssh root@192.168.1.10",
    "icon": "🖥️",
    "startingDirectory": "%USERPROFILE%"
}
```

Now open a Proxmox SSH session directly from the profile dropdown — no need to type the SSH command each time.

---

## Related guides

- [PowerShell – Getting Started](/guides/operating-systems/windows/powershell-getting-started/) — start with PowerShell basics
- [Install WSL2 and Kali Linux on Windows 11](/guides/operating-systems/windows/install-wsl2-kali-linux/) — run Linux in Windows Terminal
- [SSH to Windows – Remote Access](/guides/operating-systems/windows/ssh-windows-remote-access/) — SSH into Windows from other machines
- [Essential CMD Commands](/guides/operating-systems/windows/essential-cmd-commands/) — CMD reference alongside PowerShell
