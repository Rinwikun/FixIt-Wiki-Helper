# Korn Shell (ksh) Cheat Sheet

## Problem / Context

The Korn Shell (`ksh`) blends Bourne shell compatibility with C-shell-inspired interactive features (command history, aliases, job control) and adds its own advanced scripting constructs — many of which were later adopted by Bash. `ksh` remains the default or preferred shell in some enterprise Unix environments (notably AIX and some Solaris/HP-UX systems). This cheat sheet focuses on `ksh`-specific syntax and features that diverge from or predate Bash (prompt style: `$`).

**Note:** Standard CLI utilities (`ls`, `cp`, `grep`, `ps`, `chmod`, etc.) and most POSIX constructs (`if`, `for`, `[ ]`) work the same as in [Bourne Shell (sh)](./sh-cheatsheet.md). This document covers what `ksh` adds or does differently from Bash.

## Root Cause

Not applicable — this is a reference guide rather than a troubleshooting document.

## Resolution / Steps

### Variables & Typing

| Syntax | Description |
|---|---|
| `typeset -i x=5` | Declare a variable as an integer (enables arithmetic without `$(( ))` in some contexts) |
| `typeset -r CONST=10` | Declare a read-only (constant) variable |
| `typeset -u VAR` | Force a variable's value to uppercase on assignment |
| `typeset -l VAR` | Force a variable's value to lowercase on assignment |
| `print -v VAR` | Print a variable using `ksh`'s native `print` built-in |

### Output: `print` vs `echo`

| Syntax | Description |
|---|---|
| `print "text"` | `ksh`'s preferred, more consistent output built-in (behaves predictably across `ksh` versions, unlike `echo`) |
| `print -n "text"` | Print without a trailing newline |
| `print -- "-text"` | Print text that starts with a dash safely (`--` signals end of options) |

### Arrays

| Syntax | Description |
|---|---|
| `set -A arr one two three` | Declare an indexed array (ksh88 syntax) |
| `arr=(one two three)` | Declare an indexed array (ksh93+ syntax, closer to Bash) |
| `echo ${arr[0]}` | Access the first element (0-indexed, like Bash) |
| `typeset -A assoc` | Declare an **associative array** (ksh93+) |
| `assoc[key]="value"` | Assign a key-value pair in an associative array |
| `echo ${assoc[key]}` | Access a value by key |

### Functions

```ksh
function greet {
    print "Hello, $1"
}

# POSIX-style alternative (also valid in ksh)
greet2() {
    print "Hi, $1"
}

greet "World"
```

### Conditionals & Arithmetic

| Syntax | Description |
|---|---|
| `[[ "$a" == "$b" ]]` | Extended test syntax (ksh introduced this before Bash adopted it) |
| `(( a + b ))` | Arithmetic evaluation (also predates Bash's adoption of the same syntax) |
| `(( result = a + b ))` | Arithmetic assignment |

### `select` Menus

```ksh
select option in "Start" "Stop" "Exit"; do
    case $option in
        "Start") print "Starting..." ;;
        "Stop")  print "Stopping..." ;;
        "Exit")  break ;;
    esac
done
```

`select` generates a numbered menu automatically from the given list and prompts for a choice — useful for interactive scripts without manually building a menu loop.

### Job Control

| Command | Description |
|---|---|
| `command &` | Run a command in the background |
| `jobs` | List background jobs in the current session |
| `fg %1` | Bring job 1 to the foreground |
| `bg %1` | Resume job 1 in the background |
| `wait` | Wait for all background jobs to finish |

### Configuration Files

| File | Purpose |
|---|---|
| `~/.kshrc` | Interactive shell config (aliases, functions, prompt) — must be referenced via the `ENV` environment variable to be auto-loaded |
| `export ENV=~/.kshrc` | Common line in `.profile` to activate `.kshrc` loading for interactive sessions |
| `~/.profile` | Login shell config (environment variables, PATH) |

---

## Quick Diagnostic Commands

| Command | Purpose |
|---|---|
| `echo $KSH_VERSION` | Confirm the Korn Shell version (ksh93 and later only) |
| `ksh -n script.ksh` | Syntax-check a script without executing it |
| `ksh -x script.ksh` | Run a script with execution tracing (debug mode) |
| `whence <command>` | `ksh`'s native equivalent of `which`/`type` — shows how a command name resolves |

## Prevention Tips

- Prefer `print` over `echo` in `ksh` scripts — `echo`'s handling of backslash escapes and flags varies across `ksh` versions and systems, while `print` behaves consistently.
- Confirm whether the target system runs ksh88 or ksh93 before using associative arrays or `arr=(...)` syntax — ksh88 (still found on some older AIX/Solaris systems) only supports the older `set -A` array syntax.
- When portability across Bourne-family shells matters, stick to POSIX-compatible constructs shared with [sh](./sh-cheatsheet.md) rather than ksh-specific extensions like `typeset` or `select`.
