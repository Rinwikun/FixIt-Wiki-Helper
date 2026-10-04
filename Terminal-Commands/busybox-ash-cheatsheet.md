# BusyBox ash Cheat Sheet

## Problem / Context

BusyBox is a single compact binary that bundles stripped-down implementations of dozens of standard Unix utilities (`ls`, `grep`, `ps`, `wget`, etc.) plus its own shell, `ash` (a descendant of the original Almquist Shell, the same lineage Dash comes from). It is the default userland on **Alpine Linux** — the most common minimal base image for Docker containers — and is also widely used on embedded Linux systems and routers. This cheat sheet covers BusyBox `ash`'s syntax and the practical quirks of working in a BusyBox-based environment (prompt style: `/ #` as root in a container, or `$`).

**Note:** BusyBox `ash` syntax is POSIX `sh`-compatible — see [Bourne Shell (sh) Cheat Sheet](./sh-cheatsheet.md) for the full scripting syntax reference. This document focuses on what's specific to the BusyBox environment itself: its applet model, Alpine's package manager, and common gotchas when a script assumes a full GNU/Bash toolset.

## Root Cause

Not applicable — this is a reference guide rather than a troubleshooting document.

## Resolution / Steps

### Understanding BusyBox's "Applet" Model

BusyBox implements many commands as **applets** inside one binary, symlinked to their familiar names:

```sh
# List every applet BusyBox provides in this build
busybox --list

# Confirm a command is actually the BusyBox applet, not a separate binary
ls -l $(which ls)
# Typical output: ... /bin/ls -> busybox
```

Each applet implements only a **subset** of its GNU counterpart's flags — scripts written and tested against GNU coreutils (common on Debian/Ubuntu/macOS) can fail silently or reject flags inside an Alpine container.

### Confirming the Shell Itself

```sh
readlink -f /bin/sh
# Typical Alpine output: /bin/busybox (ash applet)

echo $0
```

### Common Flag/Behavior Differences from GNU Tools

| Command | GNU (Debian/Ubuntu) | BusyBox (Alpine) | Note |
|---|---|---|---|
| `grep --color` | Supported | Often unsupported | Omit color flags in portable scripts |
| `sed -i` | Supported | Supported, but **requires a suffix argument on some builds** (`sed -i '' ...` vs `sed -i ...`) | Verify behavior before relying on in-place edits |
| `date -d` | Full GNU date-string parsing | Limited or unsupported in some BusyBox builds | Use POSIX-portable date arithmetic instead |
| `ps aux` | Full BSD-style output | Reduced columns/flags | Use `ps` (no flags) or `ps -ef` for safer baseline output |

### Package Management (Alpine's `apk`, not `apt`)

| Command | Description |
|---|---|
| `apk update` | Refresh the local package index |
| `apk add <package>` | Install a package |
| `apk add --no-cache <package>` | Install without caching the index locally (standard practice in Dockerfiles to keep image size down) |
| `apk del <package>` | Remove a package |
| `apk info` | List installed packages |

### Common Gotcha: Missing Tools in Minimal Images

Alpine's minimal base image does **not** include Bash, `curl`, or many common utilities by default:

```sh
# bash: not found — this is expected on a fresh alpine image
bash

# Install bash explicitly if a script genuinely requires it
apk add --no-cache bash

# curl is often absent too — BusyBox provides a minimal wget instead
wget -qO- https://example.com
```

**Better practice:** rewrite the script to be POSIX `sh`-compatible (see [sh cheat sheet](./sh-cheatsheet.md)) rather than installing Bash just to run a Bash-specific script — this keeps the image smaller, which is the entire reason Alpine/BusyBox is chosen in the first place.

### `ash` vs `hush`

Some extremely space-constrained embedded BusyBox builds use `hush` instead of `ash` — an even more minimal shell with fewer features (no job control, limited built-ins). If a script behaves unexpectedly on an embedded device, confirm which shell variant is actually compiled in:

```sh
busybox | head -1
```

---

## Quick Diagnostic Commands

| Command | Purpose |
|---|---|
| `busybox --list` | List every applet available in this BusyBox build |
| `readlink -f /bin/sh` | Confirm `/bin/sh` resolves to BusyBox |
| `ash -n script.sh` | Syntax-check a script without executing it |
| `apk info` | List installed Alpine packages |
| `cat /etc/os-release` | Confirm the base image is actually Alpine (vs. another BusyBox-based distro) |

## Prevention Tips

- When writing a Dockerfile `ENTRYPOINT`/`CMD` script for an Alpine-based image, test it **inside the actual container**, not just on the host machine — a script that works fine under Bash on a dev laptop can fail silently under BusyBox `ash`.
- Run `checkbashisms` (see [Dash cheat sheet](./dash-cheatsheet.md)) against scripts intended for Alpine containers — the same bashism pitfalls that break Dash also break BusyBox `ash`.
- Prefer multi-stage Docker builds where a full-featured Bash/GNU-based image handles complex build logic, and the final Alpine-based runtime image only needs minimal `ash`-compatible startup commands.
