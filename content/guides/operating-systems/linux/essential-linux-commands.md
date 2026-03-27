---
title: Essential Linux Commands
description: A practical reference guide for the Linux commands you will actually use every day — navigation, files, permissions, networking, and system management.
date: 2026-03-27
tags: [linux, terminal, commands, bash, reference]
---

{{< callout type="warning" >}}
**Use at your own risk.** All guides and scripts are provided for educational purposes only. Always review and understand any code before running it — especially with administrative privileges. Test in a safe environment before using in production. Your system, your responsibility.
{{< /callout >}}

This is a practical reference for the Linux commands you will actually use day to day. Not an exhaustive list — just the ones that matter most when working with servers, home labs, and infrastructure.

---

## Navigation

| Command | What it does |
|---|---|
| `pwd` | Print current directory |
| `ls` | List files and folders |
| `ls -la` | List all files with details and hidden files |
| `cd /path/to/folder` | Change directory |
| `cd ..` | Go up one level |
| `cd ~` | Go to home directory |
| `cd -` | Go back to previous directory |

```bash
# Example
ls -la /etc/
```

---

## Files and Directories

| Command | What it does |
|---|---|
| `mkdir foldername` | Create a directory |
| `mkdir -p a/b/c` | Create nested directories |
| `touch file.txt` | Create an empty file |
| `cp file.txt /destination/` | Copy a file |
| `cp -r folder/ /destination/` | Copy a folder recursively |
| `mv file.txt /destination/` | Move or rename a file |
| `rm file.txt` | Delete a file |
| `rm -rf folder/` | Delete a folder and all contents |
| `cat file.txt` | Print file contents |
| `less file.txt` | Scroll through file contents |
| `head -n 20 file.txt` | Show first 20 lines |
| `tail -n 20 file.txt` | Show last 20 lines |
| `tail -f file.txt` | Follow file in real time (great for logs) |

{{< callout type="warning" >}}
`rm -rf` deletes files and folders permanently with no confirmation. Double-check your path before running it.
{{< /callout >}}

---

## Searching

| Command | What it does |
|---|---|
| `grep "text" file.txt` | Search for text in a file |
| `grep -r "text" /path/` | Search recursively in a directory |
| `grep -i "text" file.txt` | Case-insensitive search |
| `find /path -name "*.log"` | Find files by name |
| `find /path -type f -mtime -7` | Find files modified in last 7 days |

```bash
# Find all .conf files under /etc
find /etc -name "*.conf"

# Search for "error" in all log files
grep -r "error" /var/log/
```

---

## Permissions

| Command | What it does |
|---|---|
| `chmod 755 file` | Set permissions (rwxr-xr-x) |
| `chmod +x script.sh` | Make a file executable |
| `chown user:group file` | Change owner and group |
| `chown -R user:group folder/` | Change ownership recursively |
| `ls -l` | View file permissions |

**Permission numbers explained:**

```
7 = rwx (read, write, execute)
6 = rw- (read, write)
5 = r-x (read, execute)
4 = r-- (read only)
0 = --- (no permissions)
```

```bash
# Make a script executable
chmod +x deploy.sh

# Give owner full access, others read only
chmod 644 config.txt
```

---

## System Information

| Command | What it does |
|---|---|
| `uname -a` | Kernel and system info |
| `hostname` | Show hostname |
| `uptime` | How long the system has been running |
| `whoami` | Current logged-in user |
| `df -h` | Disk usage (human readable) |
| `du -sh /path/` | Size of a specific folder |
| `free -h` | RAM usage |
| `top` | Real-time process monitor |
| `htop` | Better process monitor (install with apt) |
| `lscpu` | CPU information |
| `lsblk` | List block devices (disks) |

---

## Process Management

| Command | What it does |
|---|---|
| `ps aux` | List all running processes |
| `ps aux | grep nginx` | Find a specific process |
| `kill PID` | Stop a process by ID |
| `kill -9 PID` | Force kill a process |
| `pkill nginx` | Kill process by name |

```bash
# Find and kill a stuck process
ps aux | grep myapp
kill -9 12345
```

---

## Services (systemd)

| Command | What it does |
|---|---|
| `sudo systemctl status nginx` | Check service status |
| `sudo systemctl start nginx` | Start a service |
| `sudo systemctl stop nginx` | Stop a service |
| `sudo systemctl restart nginx` | Restart a service |
| `sudo systemctl enable nginx` | Start service on boot |
| `sudo systemctl disable nginx` | Disable service on boot |
| `sudo journalctl -u nginx -f` | Follow service logs live |

---

## Networking

| Command | What it does |
|---|---|
| `ip a` | Show IP addresses and interfaces |
| `ip r` | Show routing table |
| `ping google.com` | Test connectivity |
| `curl https://example.com` | Fetch a URL |
| `wget https://example.com/file` | Download a file |
| `ss -tulnp` | Show open ports and listening services |
| `nmap -sV 192.168.1.1` | Scan a host (install nmap first) |
| `traceroute google.com` | Trace network path |

```bash
# Check what is listening on port 80
ss -tulnp | grep :80
```

---

## Package Management (apt)

| Command | What it does |
|---|---|
| `sudo apt update` | Update package lists |
| `sudo apt upgrade` | Upgrade installed packages |
| `sudo apt install nginx` | Install a package |
| `sudo apt remove nginx` | Remove a package |
| `sudo apt autoremove` | Remove unused dependencies |
| `apt search nginx` | Search for a package |
| `apt show nginx` | Show package details |

---

## Text Editing

| Command | What it does |
|---|---|
| `nano file.txt` | Open file in nano editor |
| `vim file.txt` | Open file in vim editor |

**nano basics:**
- `CTRL+O` — save
- `CTRL+X` — exit
- `CTRL+W` — search

**vim basics:**
- `i` — enter insert mode
- `ESC` — exit insert mode
- `:w` — save
- `:q` — quit
- `:wq` — save and quit
- `:q!` — quit without saving

---

## Useful Shortcuts

| Shortcut | What it does |
|---|---|
| `CTRL+C` | Cancel current command |
| `CTRL+Z` | Suspend current process |
| `CTRL+L` | Clear the terminal |
| `CTRL+A` | Jump to beginning of line |
| `CTRL+E` | Jump to end of line |
| `↑ / ↓` | Navigate command history |
| `!!` | Repeat last command |
| `sudo !!` | Repeat last command with sudo |

---

## Related Links

- [SSH Keys – The Right Way](/guides/operating-systems/linux/ssh-keys-linux/) — secure your server access
- [UFW Firewall on Ubuntu and Debian](/guides/operating-systems/linux/ufw-firewall-linux/) — lock down your server
- [Linux man pages](https://man7.org/linux/man-pages/) — full documentation for any command
