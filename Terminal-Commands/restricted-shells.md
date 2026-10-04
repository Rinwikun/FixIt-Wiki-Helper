# Restricted Shells & Login-Denial Mechanisms

## Problem / Context

Some accounts should never get a full interactive shell — a service account, a Git-over-SSH user, or an SFTP-only user all need **narrower** access than a normal login. Unix provides several mechanisms for this, ranging from "restricted mode" variants of normal shells to shells that deny login outright. Misunderstanding which mechanism to use (or assuming a restricted shell alone is a strong security boundary) is a common source of either broken workflows or under-secured accounts. This document covers the main options and their real security properties.

## Root Cause

Not applicable — this is a reference/concept guide rather than a troubleshooting document.

## Resolution / Steps

### `rbash` (Restricted Bash)

Invoked as `rbash`, or `bash` with the `-r`/`--restricted` flag, or by symlinking `rbash` as a user's login shell.

**What it disables once active:**

| Restriction | Effect |
|---|---|
| `cd` | Disabled — user cannot change directories |
| Setting/unsetting `SHELL`, `PATH`, `ENV`, `BASH_ENV` | Disabled — prevents escaping via environment manipulation |
| Commands containing `/` | Disabled — user can only run commands found in `PATH`, not arbitrary binaries by full path |
| Output redirection (`>`, `>|`, `<>`, `>>`) | Disabled — prevents overwriting files outside intended scope |
| `exec` | Disabled |
| Adding new builtins via `enable` | Disabled |
| Importing function definitions from the environment | Disabled |

```bash
# Set as a user's login shell
sudo usermod -s /bin/rbash <username>
```

**⚠ Known limitation:** `rbash` (and restricted mode in other shells) is a **soft restriction**, not a hard security boundary. Many documented escape techniques exist (invoking an editor that spawns a shell, using `scp`/`rsync` with crafted arguments, etc.). It should be combined with a **restricted `PATH`** containing only a minimal, vetted set of commands, and ideally a `chroot` jail, for genuine hardening — never rely on it alone for sensitive access control.

### `rksh` / `rzsh` (Restricted Korn/Z Shell)

Same underlying concept, different shell family:

```sh
# ksh: enable restricted mode
ksh -r
# or inside a script/session:
set -o restricted
```

```zsh
# zsh: enable restricted mode
zsh -r
# or via a symlink named rzsh pointing to zsh
```

The same restrictions apply conceptually (no `cd`, no `PATH`/`SHELL` changes, no `/`-qualified commands) and the same escape-risk caveat holds.

### `git-shell` (Git-over-SSH Only)

A purpose-built restricted shell that allows **only** Git operations (`git-upload-pack`, `git-receive-pack`, `git-upload-archive`) over SSH — commonly used for Git hosting accounts that should do nothing except push/pull.

```bash
# Confirm git-shell is in the list of allowed login shells
cat /etc/shells

# Set as a user's login shell
sudo usermod -s $(which git-shell) <username>

# Optionally allow a custom whitelist of extra commands
mkdir -p ~/git-shell-commands
# Add executable scripts here — they become the only other commands
# available to this user over SSH
```

Attempting an interactive SSH login as this user is rejected with a message like `fatal: Interactive git shell is not enabled` — non-Git commands are refused outright.

### `nologin` / `/bin/false` (Deny Interactive Login Entirely)

Used for service accounts that should **never** log in interactively (e.g., `www-data`, `nobody`, application daemon accounts) but still need to exist for file ownership/process purposes.

```bash
sudo usermod -s /usr/sbin/nologin <username>
# or, equivalent effect:
sudo usermod -s /bin/false <username>
```

| | `nologin` | `/bin/false` |
|---|---|---|
| Behavior | Prints a message (e.g., "This account is currently not available.") then exits | Exits silently with no message |
| Customization | Message can sometimes be customized via `/etc/nologin.txt` (platform-dependent) | Not customizable — it's just the `false` binary |

### `sftp-server` / `scponly` (File-Transfer-Only Access)

For accounts that need **only** file transfer, not any shell access at all:

**Native OpenSSH approach (preferred on modern systems) — in `sshd_config`:**
```sshconfig
Match User <username>
    ForceCommand internal-sftp
    ChrootDirectory /home/%u
    AllowTcpForwarding no
    X11Forwarding no
```

This forces every session for that user into the built-in SFTP subsystem, jailed to their home directory — no shell is ever reached.

**`scponly`** is an older third-party restricted shell providing a similar effect (SCP/SFTP only) for systems without OpenSSH's native `ForceCommand`/`ChrootDirectory` support, though the native approach above is generally preferred today.

---

## Quick Diagnostic Commands

| Command | Purpose |
|---|---|
| `cat /etc/shells` | List shells registered as valid login shells on the system |
| `getent passwd <username>` | Check which shell is assigned to a specific user account |
| `chsh -s <shell> <username>` | Change a user's login shell |
| `grep <username> /etc/passwd` | Quick manual check of a user's shell field (7th field) |

## Prevention Tips

- Never treat `rbash`/`rksh`/`rzsh` restricted mode as a complete security boundary by itself — pair it with a minimal restricted `PATH`, no write access outside an intended directory, and ideally a `chroot` jail for anything handling untrusted users.
- Prefer `git-shell` over a generic restricted shell for Git-hosting accounts — it's purpose-built and has a much smaller attack surface than trying to lock down `rbash` to behave like a Git-only shell.
- Prefer OpenSSH's native `ForceCommand internal-sftp` + `ChrootDirectory` over third-party tools like `scponly` on modern systems — it's maintained as part of OpenSSH itself and avoids depending on an external, less frequently updated project.
