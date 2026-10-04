# WSL (Windows Subsystem for Linux) Cheat Sheet

## Problem / Context

WSL lets Windows run genuine Linux distributions — Ubuntu, Debian, Arch, etc. — with their real shells (`bash`, `zsh`, `fish`) and full package ecosystems, which is a fundamentally different approach from [Git Bash/MSYS2/Cygwin](./git-bash-msys2-cygwin-cheatsheet.md): those are *compatibility layers* translating Unix calls onto Windows, while WSL2 runs an **actual Linux kernel** in a lightweight virtual machine. This cheat sheet covers managing WSL itself from the Windows side (prompt style for WSL management commands: Windows `>` or PowerShell `PS >`; once inside a distro, the prompt is whatever that distro's shell uses).

**Note:** Once inside a WSL distro, you're using a real Bash (or whatever shell that distro defaults to) — see [Bash Cheat Sheet](./bash-cheatsheet.md) for the actual shell syntax. This document focuses on managing WSL itself: installing, listing, and switching between distros, and the Windows↔Linux filesystem/networking bridge.

## Root Cause

Not applicable — this is a reference guide rather than a troubleshooting document.

## Resolution / Steps

### Installation

```powershell
# Install WSL with the default distro (Ubuntu), from PowerShell/CMD as Administrator
wsl --install

# Install a specific distro
wsl --install -d Debian

# List distros available to install
wsl --list --online
```

### Managing Installed Distros

| Command | Description |
|---|---|
| `wsl --list --verbose` (or `wsl -l -v`) | List installed distros, their WSL version (1 or 2), and running state |
| `wsl --set-default <distro>` | Set which distro launches when running plain `wsl` |
| `wsl --set-version <distro> 2` | Convert a distro between WSL1 and WSL2 |
| `wsl --unregister <distro>` | Completely remove a distro and its data (⚠ irreversible) |
| `wsl --export <distro> <file.tar>` | Export a distro's filesystem to a tarball (backup/migration) |
| `wsl --import <distro> <install-dir> <file.tar>` | Import a distro from a previously exported tarball |

### Launching & Running Commands

| Command | Description |
|---|---|
| `wsl` | Launch the default distro's default shell |
| `wsl -d <distro>` | Launch a specific distro |
| `wsl ~` | Launch directly into the distro's home directory |
| `wsl <command>` | Run a single command inside the default distro without opening an interactive session |
| `wsl -d <distro> -- <command>` | Run a single command inside a specific distro |

### Shutting Down / Restarting

| Command | Description |
|---|---|
| `wsl --shutdown` | Shut down **all** running WSL distros and the underlying WSL2 VM |
| `wsl --terminate <distro>` | Stop a specific distro without affecting others |
| `wsl --status` | Show the default distro and default WSL version |

### Filesystem Bridging

```bash
# From inside WSL — access Windows drives
cd /mnt/c/Users/LOQ/Documents
```

```powershell
# From Windows Explorer or PowerShell — access a WSL distro's Linux filesystem
\\wsl$\Ubuntu\home\username\
```

**Performance note:** accessing Linux files from the Windows side (`\\wsl$\...`) or Windows files from the Linux side (`/mnt/c/...`) both incur a cross-filesystem performance penalty under WSL2. For heavy I/O workloads (large repos, builds), keep project files **inside** the Linux filesystem (e.g., `~/projects/`) and work from within WSL, rather than under `/mnt/c/`.

### Networking

WSL2 automatically forwards `localhost` between Windows and Linux — a server started inside WSL2 on `localhost:3000` is reachable from a browser running on Windows at `http://localhost:3000` with no extra configuration needed in most setups.

### Configuration Files

| File | Scope | Purpose |
|---|---|---|
| `%UserProfile%\.wslconfig` | Global (all distros) | WSL2 VM settings — memory limit, CPU count, swap size |
| `/etc/wsl.conf` (inside a distro) | Per-distro | Distro-specific settings — automount options, default user, systemd enablement |

**Example `.wslconfig`:**
```ini
[wsl2]
memory=4GB
processors=2
```

**Example `/etc/wsl.conf` (enabling systemd, common for running services inside WSL):**
```ini
[boot]
systemd=true
```

### VS Code Integration

```bash
# From inside a WSL distro, in a project directory
code .
```

With the **Remote - WSL** extension installed, this opens VS Code on Windows while running the actual editing/extension host inside the Linux environment — files, terminal, and extensions all operate in the Linux context.

---

## Quick Diagnostic Commands

| Command | Purpose |
|---|---|
| `wsl --status` | Show default distro and default WSL version |
| `wsl --list --verbose` | Show all installed distros and their WSL version/state |
| `wsl --version` | Show WSL component version info |
| `cat /proc/version` (inside WSL) | Confirm you're running inside the WSL Linux kernel |
| `wsl --update` | Update the WSL core components |

## Prevention Tips

- Choose WSL2 (the default on modern Windows 10/11) for most use cases — it provides a real Linux kernel with far better compatibility than WSL1, at the cost of slower cross-filesystem I/O between Windows and Linux.
- Keep active project files on the Linux filesystem side (`~/...`) rather than under `/mnt/c/...` when working primarily from within WSL — this avoids the cross-filesystem I/O penalty and is the single most common WSL performance complaint.
- Run `wsl --shutdown` if a distro becomes unresponsive or after changing `.wslconfig` — VM-level settings only take effect after the WSL2 VM is fully restarted, not just the individual distro.
