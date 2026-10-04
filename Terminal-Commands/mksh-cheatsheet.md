# mksh (MirBSD Korn Shell) Cheat Sheet

## Problem / Context

`mksh` is a lightweight, modernized continuation of the public-domain Korn Shell, and is notably the **default shell on Android** (`/system/bin/sh`), reached via `adb shell` or a terminal emulator app. It's also used on some BSD systems as a lean alternative to full `ksh93`. This cheat sheet covers `mksh` syntax and the practical commands most commonly needed in an Android shell session (prompt style: `$` or `#` if rooted).

**Note:** `mksh`'s core scripting syntax is Korn-shell-derived — see [Korn Shell (ksh) Cheat Sheet](./ksh-cheatsheet.md) for `typeset`, arrays, `[[ ]]`, and `(( ))` syntax, which largely carries over. This document focuses on what's specific to `mksh` itself and to the Android shell environment where it's most commonly encountered.

## Root Cause

Not applicable — this is a reference guide rather than a troubleshooting document.

## Resolution / Steps

### Confirming `mksh` Is the Active Shell

```sh
echo $KSH_VERSION
# Typical output includes "MIRBSD KSH" if running mksh

readlink -f /system/bin/sh
```

### Accessing the Shell on Android

| Command | Description |
|---|---|
| `adb shell` | Open an interactive `mksh` session on a connected Android device via ADB |
| `adb shell <command>` | Run a single command on the device without entering an interactive session |
| `exit` | Leave the shell session |

### Android-Specific Commands Commonly Run Inside `mksh`

These are Android platform tools, not `mksh` built-ins, but they're what the shell is used for in practice:

| Command | Description |
|---|---|
| `pm list packages` | List installed app packages |
| `pm install <file.apk>` | Install an APK (when run with appropriate permissions) |
| `am start -n <package>/<activity>` | Launch a specific app activity |
| `getprop` | List all system properties (build info, device config) |
| `getprop ro.build.version.release` | Get a specific property (e.g., Android version) |
| `settings get <namespace> <key>` | Read a system/secure/global setting value |
| `dumpsys <service>` | Dump diagnostic state for a system service (battery, activity, etc.) |
| `logcat` | Stream the live system log |
| `su` | Switch to root shell (only works on rooted devices) |

### Variables, Functions & Arrays (Korn-Derived Syntax)

```sh
# Variable assignment
VAR="value"

# Function definition
greet() {
    echo "Hello, $1"
}

# Indexed array
set -A arr one two three
echo "${arr[0]}"

# Arithmetic
(( result = 2 + 3 ))
echo $result
```

See [ksh cheat sheet](./ksh-cheatsheet.md) for the full syntax reference — `typeset`, `[[ ]]` extended tests, and `select` menus all work the same way in `mksh`.

### Configuration

| File | Purpose |
|---|---|
| `~/.mkshrc` | Interactive shell config — must be referenced via the `ENV` variable to auto-load, same pattern as `ksh` |
| `export ENV=~/.mkshrc` | Common line to activate `.mkshrc` loading |

On stock Android, there is typically no persistent home directory config loaded by default — most `adb shell` sessions start with a minimal environment.

### File System Navigation Notes (Android-Specific)

| Path | Purpose |
|---|---|
| `/sdcard` or `/storage/emulated/0` | Shared user storage (Downloads, Pictures, etc.) |
| `/data/data/<package>` | App-private data directory (requires root or run-as for access) |
| `/system` | Read-only system partition (requires root + remount to write) |

---

## Quick Diagnostic Commands

| Command | Purpose |
|---|---|
| `echo $KSH_VERSION` | Confirm `mksh` is the active shell and its version |
| `mksh -n script.sh` | Syntax-check a script without executing it |
| `mksh -x script.sh` | Run a script with execution tracing (debug mode) |
| `getprop ro.build.version.release` | Check the Android OS version on the current device |
| `whoami` | Confirm the current shell user (`shell` by default, `root` if escalated) |

## Prevention Tips

- Most stock (non-rooted) Android shells run with restricted permissions as the `shell` user — many filesystem paths (`/data/data/<package>` of other apps, `/system`) will return `Permission denied` by design, not due to a shell misconfiguration.
- When writing scripts intended to run via `adb shell` automation, test directly on-device or via `adb shell sh script.sh` — behavior can differ from a desktop Linux `mksh`/`ksh` installation due to Android's restricted environment and toolbox utilities.
- For complex scripting needs beyond what stock Android's `mksh` + minimal toolbox supports, consider Termux (a full Linux userland app for Android) rather than fighting the stock shell's limitations.
