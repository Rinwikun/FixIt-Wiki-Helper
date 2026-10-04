# Fish (Friendly Interactive Shell) Cheat Sheet

## Problem / Context

Fish is a shell designed around interactive usability out of the box — syntax highlighting, autosuggestions, and sane tab completion work without any plugin framework or configuration. It deliberately **does not** target POSIX compatibility, so its scripting syntax is distinctly different from every Bourne-family shell (sh, Bash, ksh, Zsh). This cheat sheet covers Fish's own syntax and built-ins (prompt style: `>`).

**Note:** Standard CLI utilities (`ls`, `cp`, `grep`, `ps`, `chmod`, etc.) behave identically regardless of shell. This document focuses on Fish's own scripting syntax, which is the most different from the rest of the Bourne-family shells in this collection.

## Root Cause

Not applicable — this is a reference guide rather than a troubleshooting document.

## Resolution / Steps

### Variables

| Syntax | Description |
|---|---|
| `set VAR value` | Assign a variable — **no `=` sign**, unlike every Bourne-family shell |
| `echo $VAR` | Reference a variable's value |
| `set -x VAR value` | Export a variable into the environment (equivalent to Bash's `export VAR=value`) |
| `set -e VAR` | Erase (unset) a variable |
| `set -U VAR value` | Set a **universal** variable — persists across all sessions and terminals automatically, no `.rc` file editing required |
| `set VAR val1 val2 val3` | Fish variables are inherently list-like; this creates a 3-element list in `VAR` |

### Conditionals

```fish
if test $x = 1
    echo "x is one"
else if test $x = 2
    echo "x is two"
else
    echo "x is something else"
end
```

**Note:** Fish has no `then`/`fi` — blocks are closed with a single `end` keyword across all constructs (`if`, `for`, `while`, `function`).

### Loops

```fish
# for loop
for item in one two three
    echo $item
end

# while loop
set count 0
while test $count -lt 5
    set count (math $count + 1)
end
```

### Arithmetic

| Syntax | Description |
|---|---|
| `math "2 + 3"` | Fish has no `$(( ))` arithmetic expansion — arithmetic is done via the external `math` command |
| `set result (math "$a * $b")` | Assign the result of a calculation to a variable |

### Functions

```fish
function greet
    echo "Hello, $argv[1]"
end

greet World
```

**Note:** Function arguments are accessed via the `$argv` list (1-indexed), not `$1`/`$2` positional parameters.

### Command Substitution

| Syntax | Description |
|---|---|
| `set files (ls)` | Command substitution uses parentheses `( )`, not Bash's backticks or `$( )` |
| `echo (date)` | Inline command substitution directly in an argument position |

### Autosuggestions & Completion (Built-in, No Plugins)

| Feature | Description |
|---|---|
| `→` (Right Arrow) | Accept the greyed-out autosuggestion shown inline as you type |
| `Tab` | Trigger completion — Fish auto-generates completions from command `--help`/man pages |
| `Alt+→` | Accept only the next suggested word, not the full suggestion |

### Universal Variables & Configuration

| Command / File | Description |
|---|---|
| `~/.config/fish/config.fish` | Main configuration file (aliases, abbreviations, startup commands) |
| `set -U fish_greeting ""` | Example universal variable — disables Fish's startup greeting message, persists forever |
| `funcsave <name>` | Save a function permanently to `~/.config/fish/functions/` |

### Abbreviations (Fish's Alias Alternative)

| Command | Description |
|---|---|
| `abbr -a gco git checkout` | Define an abbreviation — expands to the full text in the command line when typed + space, unlike a traditional alias |
| `abbr --show` | List all defined abbreviations |
| `abbr -e gco` | Erase an abbreviation |

### Web-Based Configuration

| Command | Description |
|---|---|
| `fish_config` | Launch a local web UI for choosing color themes, prompt styles, and reviewing functions/variables/history |

---

## Quick Diagnostic Commands

| Command | Purpose |
|---|---|
| `echo $FISH_VERSION` | Confirm the current Fish version |
| `fish -n script.fish` | Syntax-check a script without executing it |
| `functions` | List all currently defined functions |
| `type <command>` | Show whether a name is a builtin, function, or external binary |

## Prevention Tips

- Fish scripts are **not** portable to `#!/bin/sh` or Bash environments — always use `#!/usr/bin/env fish` as the shebang, and never assume a `.fish` script will run correctly under a POSIX shell.
- Use `set -U` sparingly — universal variables persist silently across every future session, which can cause confusing "it works on my machine" state if forgotten about.
- For portable automation scripts (CI/CD, cron, deployment), write in [Bourne Shell (sh)](./sh-cheatsheet.md) or [Bash](./bash-cheatsheet.md) instead — Fish is best suited for interactive daily use, not script portability.
