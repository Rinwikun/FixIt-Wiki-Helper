# Bourne Shell (sh) Cheat Sheet

## Problem / Context

`sh` refers to the POSIX-compliant Bourne Shell (or a POSIX-mode shell like `dash` on Debian/Ubuntu, or BusyBox `ash` on Alpine-based containers). It is the minimal, portable baseline used for `#!/bin/sh` scripts, system init scripts, and lightweight container images. Scripts written with Bash-specific syntax (`[[`, arrays, `echo -e`) often fail or behave unexpectedly when executed under `sh`, since not every `sh` implementation supports Bash extensions. This cheat sheet covers portable, POSIX-compliant `sh` syntax and commonly used built-ins (prompt style: `$`).

**Note:** Standard CLI utilities (`ls`, `cp`, `grep`, `ps`, `chmod`, etc.) behave identically regardless of shell — see [Bash Cheat Sheet](./bash-cheatsheet.md) for those. This document focuses on `sh`'s own scripting syntax and built-ins, and where it diverges from Bash.

## Root Cause

Not applicable — this is a reference guide rather than a troubleshooting document.

## Resolution / Steps

### Variables & Substitution

| Syntax | Description |
|---|---|
| `VAR="value"` | Assign a variable (no spaces around `=`) |
| `echo "$VAR"` | Reference a variable's value (always quote to avoid word-splitting) |
| `${VAR:-default}` | Use `default` if `VAR` is unset or empty |
| `${VAR:=default}` | Assign `default` to `VAR` if unset or empty |
| `${#VAR}` | Get the length of a variable's value |
| `${VAR#pattern}` | Remove the shortest match of `pattern` from the start of `VAR` |
| `${VAR%pattern}` | Remove the shortest match of `pattern` from the end of `VAR` |
| `` `command` `` / `$(command)` | Command substitution (prefer `$(...)` — more readable and nestable) |

### Conditionals

| Syntax | Description |
|---|---|
| `[ "$a" = "$b" ]` | POSIX string equality test (note: single `=`, not `==`) |
| `[ "$a" -eq "$b" ]` | Numeric equality test |
| `[ -f "$file" ]` | True if `$file` exists and is a regular file |
| `[ -d "$dir" ]` | True if `$dir` exists and is a directory |
| `[ -z "$VAR" ]` | True if `$VAR` is empty |
| `[ -n "$VAR" ]` | True if `$VAR` is non-empty |
| `if [ cond ]; then ... elif [ cond2 ]; then ... else ... fi` | Standard conditional block |

**Portability note:** `sh` does not support Bash's `[[ ... ]]` extended test syntax — always use single brackets `[ ... ]` for POSIX compatibility.

### Loops

```sh
# for loop
for item in one two three; do
    echo "$item"
done

# while loop
while [ "$count" -lt 5 ]; do
    count=$((count + 1))
done

# until loop
until [ -f "/tmp/ready" ]; do
    sleep 1
done
```

### Functions

```sh
my_function() {
    echo "Argument 1: $1"
    return 0
}

my_function "hello"
```

**Note:** `sh` functions are always defined with `name() { ... }` — the `function` keyword (used in Bash) is not POSIX-standard and may be rejected by strict `sh` implementations like `dash`.

### Arithmetic

| Syntax | Description |
|---|---|
| `$((a + b))` | Perform integer arithmetic |
| `expr "$a" + "$b"` | Legacy arithmetic via external `expr` command (pre-POSIX-arithmetic fallback) |

### I/O Redirection & Pipes

| Syntax | Description |
|---|---|
| `command > file` | Redirect stdout to a file (overwrite) |
| `command >> file` | Redirect stdout to a file (append) |
| `command 2> file` | Redirect stderr to a file |
| `command > file 2>&1` | Redirect both stdout and stderr to the same file |
| `command1 \| command2` | Pipe stdout of `command1` into stdin of `command2` |
| `command < file` | Redirect a file's content as stdin |

### Exit Codes & Script Control

| Syntax | Description |
|---|---|
| `$?` | Exit code of the last executed command (`0` = success) |
| `exit 1` | Exit the script with a specific status code |
| `command1 && command2` | Run `command2` only if `command1` succeeds |
| `command1 \|\| command2` | Run `command2` only if `command1` fails |
| `set -e` | Exit the script immediately if any command fails |
| `trap 'cleanup' EXIT` | Run a cleanup function automatically on script exit |

### Positional Arguments

| Syntax | Description |
|---|---|
| `$0` | The script's name |
| `$1`, `$2`, ... | Positional arguments passed to the script |
| `$#` | Number of positional arguments |
| `$@` | All positional arguments as separate words |
| `shift` | Shift positional arguments left by one (discard `$1`) |

---

## Quick Diagnostic Commands

| Command | Purpose |
|---|---|
| `echo $0` | Confirm which shell is actually interpreting the script |
| `which sh` / `readlink -f $(which sh)` | Check which binary `sh` actually points to (often symlinked to `dash` or `bash`) |
| `sh -n script.sh` | Syntax-check a script without executing it |
| `sh -x script.sh` | Run a script with execution tracing (debug mode) |

## Prevention Tips

- Never assume `#!/bin/sh` behaves like Bash — on Debian/Ubuntu it's typically symlinked to `dash`, which strictly rejects Bash-only syntax (`[[`, arrays, `echo -e`, `function` keyword).
- Use `shellcheck script.sh` to catch non-portable syntax before deploying scripts meant to run under `sh`.
- Always quote variable expansions (`"$VAR"`) to prevent word-splitting and glob expansion surprises — this matters even more in `sh` than Bash, since `sh` has fewer safety nets.
