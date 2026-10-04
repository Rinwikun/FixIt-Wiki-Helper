# Git Bash / MSYS2 / Cygwin Cheat Sheet

## Problem / Context

Git Bash, MSYS2, and Cygwin all provide a Bash-compatible Unix-like environment on **Windows**, but they differ in scope, completeness, and how they bridge Windows paths/tools to Unix conventions. Confusing the three — or assuming full GNU/Linux behavior — is a common source of path errors, missing commands, and permission confusion on Windows dev machines. This cheat sheet covers their differences and Windows-specific bridging commands (prompt style: `$` — standard Bash prompt).

**Note:** Once inside any of these three, the actual shell syntax **is Bash** — see [Bash Cheat Sheet](./bash-cheatsheet.md) for the full command/scripting reference. This document focuses on what's unique to running Bash *on Windows* through each of these layers.

## Root Cause

Not applicable — this is a reference guide rather than a troubleshooting document.

## Resolution / Steps

### Comparing the Three

| Aspect | Git Bash | MSYS2 | Cygwin |
|---|---|---|---|
| Primary purpose | Minimal Bash bundled with Git for Windows | Full Unix-like build environment + package manager | Full POSIX compatibility layer (DLL-based) |
| Package manager | None built-in | `pacman` (same as Arch Linux) | `setup-x86_64.exe` GUI installer |
| Completeness | Minimal — Git, Bash, core utils only | Broad — build toolchains, dev libraries | Broadest — thousands of ported Unix packages |
| Typical use case | Just need Git + basic Bash on Windows | Compiling native Windows software with Unix-style tooling | Running near-complete Unix software on Windows |

### Path Translation Differences

Windows paths are represented differently across the three:

```bash
# Git Bash / MSYS2 style
/c/Users/LOQ/Documents

# Cygwin style
/cygdrive/c/Users/LOQ/Documents
```

Convert between Windows and Unix-style paths:

```bash
# Cygwin — convert a Windows path to its Unix equivalent
cygpath -u "C:\Users\LOQ\Documents"

# Cygwin — convert back to Windows style
cygpath -w /cygdrive/c/Users/LOQ/Documents

# MSYS2 has an equivalent cygpath utility as well
cygpath -u "C:\Users\LOQ"
```

### Package Management

**MSYS2 (`pacman`):**
```bash
pacman -Syu                  # Update package database and upgrade all packages
pacman -S <package>          # Install a package
pacman -R <package>          # Remove a package
pacman -Ss <search-term>     # Search available packages
```

**Cygwin:** Packages are managed through the graphical `setup-x86_64.exe` installer (re-run it to add/remove packages), though the community `apt-cyg` script exists as an unofficial CLI wrapper.

**Git Bash:** No package manager — it ships only what Git for Windows bundles. For anything beyond basic Bash + Git + core utils, use MSYS2 or Cygwin instead.

### Common Gotchas

**No `sudo` in Git Bash/MSYS2 by default:**
```bash
# This fails — sudo isn't present
sudo apt install something

# Git Bash has no package manager at all (see above)
# MSYS2: just don't prefix with sudo — pacman runs directly;
# if running as a standard user without permission, open the terminal as Administrator instead
```

**Interactive console programs may hang or render incorrectly in Git Bash:**
```bash
# Some interactive CLI tools (e.g., certain Python REPLs, SSH password prompts)
# need winpty to behave correctly inside Git Bash's MinTTY terminal
winpty python
winpty ssh user@host
```

**Line ending differences (CRLF vs LF):**
```bash
# Check Git's line-ending conversion setting — a common source of
# "file changed" noise in git status on Windows
git config --get core.autocrlf

# Common Windows-appropriate setting: convert LF to CRLF on checkout,
# CRLF back to LF on commit
git config --global core.autocrlf true
```

### Clipboard Integration (Windows-Specific)

```bash
# Git Bash / MSYS2 — pipe output to the Windows clipboard
echo "hello" | clip

# Read the Windows clipboard into a variable (MSYS2/Cygwin, if `xclip`-equivalent is installed)
```

### SSH Agent Setup (Common Pain Point on Windows)

```bash
# Start the SSH agent in a Git Bash session (not persistent across terminal restarts by default)
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

---

## Quick Diagnostic Commands

| Command | Purpose |
|---|---|
| `uname -s` | Identify which environment is active (`MINGW64_NT-...` for Git Bash, `MSYS_NT-...` for MSYS2, `CYGWIN_NT-...` for Cygwin) |
| `cygpath -u "<windows-path>"` | Convert a Windows path to Unix style |
| `which <command>` | Confirm whether a command resolves to a bundled Unix tool or falls through to a Windows `.exe` |
| `git config --get core.autocrlf` | Check the current line-ending conversion behavior |

## Prevention Tips

- Don't assume Git Bash has the same tool coverage as a real Linux box — scripts relying on common GNU utilities beyond Git/core Bash will fail with "command not found" until MSYS2 or Cygwin (or WSL) is used instead.
- Pick the right tool for the job: Git Bash for quick Git/Bash needs, MSYS2 for compiling native Windows software with a Unix-style toolchain, Cygwin for running more complete ported Unix applications, and **WSL** (a real Linux kernel environment, not a compatibility layer) when full Linux behavior is required — see a dedicated WSL note if deeper Linux compatibility becomes necessary.
- Keep `core.autocrlf` consistent across a team's Windows and Linux/macOS machines to avoid noisy whitespace-only diffs in Git.
