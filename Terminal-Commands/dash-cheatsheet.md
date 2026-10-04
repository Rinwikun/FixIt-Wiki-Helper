# Dash (Debian Almquist Shell) Cheat Sheet

## Problem / Context

Dash is a lightweight, strictly POSIX-compliant shell and is the actual binary behind `/bin/sh` on Debian and Ubuntu systems (`/bin/sh` is symlinked to `dash`, not `bash`). It is also used for `/etc/init.d` scripts, boot-time scripts, and `dpkg`/`apt` package maintainer scripts (`postinst`, `preinst`), where fast startup time matters more than interactive convenience. This cheat sheet covers Dash's scope and the practical implications of it being the system default `sh` (prompt style: `$`).

**Note:** Dash's scripting syntax **is** POSIX `sh` syntax — see [Bourne Shell (sh) Cheat Sheet](./sh-cheatsheet.md) for the full syntax reference (variables, conditionals, loops, functions). This document focuses on what's specific to Dash as an implementation, not syntax already covered there.

## Root Cause

Not applicable — this is a reference guide rather than a troubleshooting document.

## Resolution / Steps

### Confirming Dash Is Your System's `/bin/sh`

```bash
ls -l /bin/sh
# Typical Debian/Ubuntu output:
# lrwxrwxrwx 1 root root 4 ... /bin/sh -> dash
```

```bash
readlink -f /bin/sh
```

This matters because any script starting with `#!/bin/sh` on these systems runs under Dash, **not** Bash — even though many developers write `#!/bin/sh` scripts while mentally assuming Bash behavior.

### Why Dash Exists (Performance Rationale)

| Aspect | Dash | Bash |
|---|---|---|
| Startup time | Significantly faster — minimal feature set, smaller binary | Slower — loads more built-in features |
| Feature set | Strict POSIX only | POSIX + many Bash-only extensions |
| Typical use case | System/boot scripts, package scripts, performance-sensitive automation | Interactive use, feature-rich scripting |

Debian switched `/bin/sh` from Bash to Dash specifically to speed up boot time, since dozens of init scripts are invoked sequentially at startup — Dash's lighter weight meaningfully reduces cumulative overhead.

### Common Pitfall: Bashisms in `#!/bin/sh` Scripts

Scripts that declare `#!/bin/sh` but use Bash-only syntax will **fail or silently misbehave** under Dash:

```sh
# ❌ Fails under dash — [[ is a Bash extension, not POSIX
if [[ "$a" == "$b" ]]; then ...

# ✅ POSIX-compliant — works under dash and bash
if [ "$a" = "$b" ]; then ...
```

```sh
# ❌ Fails under dash — arrays are a Bash extension
arr=(one two three)

# ❌ Fails under dash — echo -e flag is not POSIX-standard
echo -e "line1\nline2"

# ❌ Fails under dash — process substitution is a Bash extension
diff <(cmd1) <(cmd2)
```

See [Bourne Shell (sh) Cheat Sheet](./sh-cheatsheet.md) for the full portable (Dash-safe) syntax reference.

### Detecting Bashisms Automatically

```bash
# Install the checker (Debian/Ubuntu)
sudo apt install devscripts

# Scan a script for non-POSIX constructs
checkbashisms script.sh
```

This is the standard tool Debian package maintainers use to verify `postinst`/`prerm`/etc. scripts are safe to run under Dash before upload.

### Explicitly Invoking Dash

| Command | Description |
|---|---|
| `dash script.sh` | Run a script explicitly under Dash, regardless of its shebang |
| `dash -n script.sh` | Syntax-check a script without executing it |
| `dash -x script.sh` | Run a script with execution tracing (debug mode) |

### Switching the System Default (Not Generally Recommended)

```bash
sudo dpkg-reconfigure dash
```

Presents a prompt to set `/bin/sh` back to Bash. **Caution:** many system scripts are written and tested specifically against Dash's stricter behavior — switching this system-wide can surface previously-masked script bugs elsewhere.

---

## Quick Diagnostic Commands

| Command | Purpose |
|---|---|
| `readlink -f /bin/sh` | Confirm what `/bin/sh` actually points to on the current system |
| `checkbashisms <script>` | Scan a script for non-POSIX (Bash-only) constructs |
| `dash -n <script>` | Syntax-check a script against Dash specifically |
| `echo $0` | Confirm which shell is actually interpreting the current script |

## Prevention Tips

- Never write `#!/bin/sh` and then test the script only under Bash (`bash script.sh`) — this masks bashisms that will fail the moment the script actually runs under `sh` on a Debian/Ubuntu system or inside a minimal Docker image.
- If a script genuinely needs Bash features, declare `#!/bin/bash` explicitly rather than `#!/bin/sh` — don't rely on the shebang being "close enough."
- Run `checkbashisms` as a pre-commit or CI step for any script shipped with `#!/bin/sh`, especially for Docker `ENTRYPOINT`/`CMD` scripts targeting Alpine (BusyBox `ash`) or Debian-based (`dash`) base images.
