# Nushell (nu) Cheat Sheet

## Problem / Context

Nushell is a cross-platform shell built around a fundamentally different model from every other shell in this collection: instead of piping raw text between commands, every command outputs **structured data** (tables, records, lists) that the next command in the pipeline can filter, sort, and transform directly — no `awk`/`sed`/`cut` text-wrangling required. This cheat sheet covers Nushell's core syntax (prompt style: `>`).

**Note:** Because Nushell's entire pipeline model differs from Bourne-family shells, there is minimal syntax overlap with [Bash](./bash-cheatsheet.md) or [sh](./sh-cheatsheet.md) — this document stands largely on its own, though the underlying concept (structured object pipelines) is conceptually similar to [PowerShell](./powershell-cheatsheet.md).

## Root Cause

Not applicable — this is a reference guide rather than a troubleshooting document.

## Resolution / Steps

### Structured Data Basics

```nu
# ls returns a table, not plain text — columns are queryable
ls

# Filter the table by a column condition
ls | where size > 10mb

# Sort the table by a column
ls | sort-by modified

# Select only specific columns
ls | select name size
```

### Variables

| Syntax | Description |
|---|---|
| `let x = 5` | Declare an **immutable** variable (default and preferred) |
| `mut x = 5` | Declare a **mutable** variable |
| `$x = 10` | Reassign a mutable variable (only valid if declared with `mut`) |
| `$env.VAR` | Access an environment variable |
| `$env.VAR = "value"` | Set an environment variable for the current session |

### Working with Structured Formats

```nu
# Open a file — Nushell auto-detects and parses the format into a table
open data.json
open data.csv
open config.toml

# Convert between formats
open data.csv | to json
open data.json | to csv

# Query a specific field from parsed structured data
open package.json | get version
```

### Pipeline Filtering & Transformation

| Command | Description |
|---|---|
| `where <condition>` | Filter rows by a condition (Nushell's equivalent of `grep`/`awk` filtering, but type-aware) |
| `sort-by <column>` | Sort rows by a column's value |
| `select <columns>` | Keep only the specified columns |
| `get <field>` | Extract a specific field/column's value(s) |
| `each { \|row\| ... }` | Apply a block of logic to each row in a table |
| `first <n>` / `last <n>` | Return the first/last N rows |
| `length` | Count the number of rows/items |

### Custom Commands (Functions)

```nu
def greet [name: string] {
    print $"Hello, ($name)"
}

greet "World"
```

String interpolation uses `$"...(expression)..."` syntax — note the parentheses, not Bash's `${VAR}` or Fish's direct `$VAR` substitution.

### Conditionals & Loops

```nu
# if/else
if $x > 10 {
    print "big"
} else {
    print "small"
}

# for loop
for item in [1 2 3] {
    print $item
}

# while loop
mut count = 0
while $count < 5 {
    $count = $count + 1
}
```

### Running External Commands

```nu
# Nushell runs external binaries normally, but their output is plain text
# unless explicitly parsed into structured data
^ls -la          # The ^ forces calling the external `ls` binary, bypassing Nushell's built-in `ls`
git status       # External commands work as expected
```

### Configuration

| File | Purpose |
|---|---|
| `~/.config/nushell/config.nu` | Main interactive configuration (aliases, keybindings, prompt) |
| `~/.config/nushell/env.nu` | Environment variable setup, loaded before `config.nu` |

### Aliases

```nu
alias ll = ls -la
```

---

## Quick Diagnostic Commands

| Command | Purpose |
|---|---|
| `version` | Show the current Nushell version and build info |
| `which <command>` | Show how a command name resolves (built-in, alias, or external) |
| `help commands` | List all built-in Nushell commands |
| `help <command>` | Show usage and examples for a specific command |
| `nu -c "script"` | Run a one-off Nushell command/script from outside an interactive session |

## Prevention Tips

- Don't expect existing Bash/Zsh scripts to run unmodified under Nushell — the structured-pipeline model is a deliberate, non-POSIX redesign, not a Bash-compatible shell.
- Use `^command` when a Nushell built-in name collides with an external binary you specifically want to invoke (e.g., `^ls` to force the system `ls` instead of Nushell's structured `ls`).
- Prefer `open` over manually parsing file content with text tools — Nushell's format auto-detection (JSON, CSV, TOML, YAML) eliminates most of the `jq`/`awk` scripting that other shells require for the same task.
