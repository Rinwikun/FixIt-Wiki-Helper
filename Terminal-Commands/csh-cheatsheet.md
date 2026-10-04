# C Shell (csh / tcsh) Cheat Sheet

## Problem / Context

`csh` (C Shell) and its improved successor `tcsh` use a syntax deliberately modeled after the C programming language rather than the Bourne shell family. This makes the scripting syntax diverge sharply from Bash/sh/ksh/zsh. `tcsh` is a strict superset of `csh`, adding command-line editing, filename/command completion, and spelling correction — most systems that ship `csh` actually symlink it to `tcsh`. This cheat sheet covers `csh`/`tcsh` syntax and built-ins (prompt style: `%` or a themed prompt).

**Note:** Standard CLI utilities (`ls`, `cp`, `grep`, `ps`, `chmod`, etc.) behave identically regardless of shell. This document focuses on `csh`/`tcsh`'s own scripting syntax, which differs the most from every other shell in this collection.

## Root Cause

Not applicable — this is a reference guide rather than a troubleshooting document.

## Resolution / Steps

### Variables & Environment

| Syntax | Description |
|---|---|
| `set var = value` | Assign a local shell variable (spaces around `=` are conventional, unlike Bourne-family shells) |
| `echo $var` | Reference a variable's value |
| `setenv VAR value` | Set an **environment** variable (equivalent to `export VAR=value` in Bash) — note: no `=` sign |
| `unset var` | Remove a local variable |
| `unsetenv VAR` | Remove an environment variable |
| `set path = (/usr/bin /usr/local/bin)` | Set the `PATH` as a space-separated list (not colon-separated like Bash) |

### Conditionals

```csh
if ($x == 1) then
    echo "x is one"
else if ($x == 2) then
    echo "x is two"
else
    echo "x is something else"
endif
```

**Note:** Blocks close with `endif`, `endsw`, and `end` (not `fi`/`esac`/`done` as in Bourne-family shells).

### Loops

```csh
# foreach loop
foreach item (one two three)
    echo $item
end

# while loop
set count = 0
while ($count < 5)
    @ count++
end
```

### Arithmetic

| Syntax | Description |
|---|---|
| `@ result = 2 + 3` | Perform integer arithmetic and assign to `result` (the `@` built-in is required) |
| `@ count++` | Increment a variable |

### Command History & Substitution

| Syntax | Description |
|---|---|
| `!!` | Re-run the previous command |
| `!$` | Reuse the last argument of the previous command |
| `!n` | Re-run history entry number `n` |
| `history` | List command history |
| `^old^new` | Repeat the previous command, replacing `old` with `new` |

### Aliases

| Syntax | Description |
|---|---|
| `alias ll "ls -la"` | Define an alias (note: quotes around the command, unlike Bash's `alias ll='ls -la'`) |
| `unalias ll` | Remove an alias |

### tcsh-Specific Enhancements

| Feature | Description |
|---|---|
| `Tab` | Filename/command completion (not available in plain `csh`) |
| `Ctrl+D` | Trigger completion listing at any point in the command line |
| `bindkey` | View or customize key bindings for command-line editing |
| `set correct = cmd` | Enable automatic spelling correction for mistyped commands |
| `set autolist` | Show possible completions automatically on ambiguous Tab press |

### Configuration Files

| File | Purpose |
|---|---|
| `~/.cshrc` / `~/.tcshrc` | Loaded for every new shell instance (interactive config, aliases) |
| `~/.login` | Loaded once at login (environment setup) |
| `~/.logout` | Run on logout |

---

## Quick Diagnostic Commands

| Command | Purpose |
|---|---|
| `echo $shell` | Confirm which shell binary is active |
| `which csh` / `which tcsh` | Check if `csh` is symlinked to `tcsh` |
| `csh -n script.csh` | Syntax-check a script without executing it |
| `csh -x script.csh` | Run a script with execution tracing (debug mode) |

## Prevention Tips

- Avoid writing portable automation scripts in `csh`/`tcsh` — this is a long-documented anti-pattern (see Tom Christiansen's *"Csh Programming Considered Harmful"*) due to inconsistent error handling, quoting pitfalls, and limited string manipulation compared to Bourne-family shells.
- Prefer `tcsh` over plain `csh` for **interactive** use — the completion and correction features meaningfully improve day-to-day usability, with full syntax compatibility.
- For any script-based automation, use [Bourne Shell (sh)](./sh-cheatsheet.md) or [Bash](./bash-cheatsheet.md) instead — nearly all modern tooling, CI/CD systems, and documentation assume Bourne-family syntax.
