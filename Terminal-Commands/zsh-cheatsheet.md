# Zsh (Z Shell) Cheat Sheet

## Problem / Context

Zsh is a Bash-compatible shell with significant quality-of-life enhancements — advanced globbing, superior tab-completion, plugin frameworks (Oh My Zsh), and richer prompt customization. It is the default shell on macOS and a common choice on Linux dev machines. This cheat sheet focuses on Zsh-specific features and syntax that diverge from Bash (prompt style: `%` or a themed prompt via Oh My Zsh).

**Note:** Standard CLI utilities (`ls`, `cp`, `grep`, `ps`, `chmod`, etc.) and most Bash scripting constructs (`if`, `for`, `[[ ]]`) work identically in Zsh — see [Bash Cheat Sheet](./bash-cheatsheet.md) for those. This document covers what Zsh adds or does differently.

## Root Cause

Not applicable — this is a reference guide rather than a troubleshooting document.

## Resolution / Steps

### Configuration Files

| File | Purpose |
|---|---|
| `~/.zshrc` | Main interactive shell config — aliases, prompt, plugins (loaded per interactive session) |
| `~/.zprofile` | Login-shell config — environment variables, PATH setup (loaded once at login) |
| `~/.zshenv` | Loaded for every Zsh invocation, including scripts — use sparingly |

### Advanced Globbing

| Syntax | Description |
|---|---|
| `**/*.txt` | Recursive glob — match `.txt` files in the current directory and all subdirectories |
| `*.txt(.)` | Match only regular files ending in `.txt` (qualifier `.` = regular file) |
| `*(/)` | Match only directories in the current path |
| `*(.om[1])` | Match the single most recently modified regular file |
| `setopt EXTENDED_GLOB` | Enable extended glob qualifiers like `^` (negation) and `#` (repetition) |

### Arrays

| Syntax | Description |
|---|---|
| `arr=(one two three)` | Declare an array |
| `echo $arr[1]` | Access the **first** element — Zsh arrays are **1-indexed**, unlike Bash (0-indexed) |
| `echo ${#arr}` | Get the number of elements in an array |
| `echo $arr` | Print all array elements |
| `arr+=(four)` | Append an element to an array |

### Autocompletion

| Command | Description |
|---|---|
| `autoload -Uz compinit && compinit` | Initialize Zsh's advanced completion system (typically placed in `.zshrc`) |
| `Tab` | Trigger completion (shows menu of options, cyclable with repeated Tab) |
| `zstyle ':completion:*' matcher-list 'm:{a-z}={A-Z}'` | Enable case-insensitive tab completion |

### Directory Navigation Enhancements

| Command | Description |
|---|---|
| `setopt AUTO_CD` | Allows typing just a directory name (no `cd`) to navigate into it |
| `pushd <dir>` | Change directory while pushing the current one onto the directory stack |
| `popd` | Return to the previous directory from the stack |
| `dirs -v` | List the directory stack with index numbers |
| `cd -<N>` | Jump to the Nth directory in the stack |

### History Management

| Setting / Command | Description |
|---|---|
| `setopt SHARE_HISTORY` | Share command history live across all open terminal sessions |
| `setopt HIST_IGNORE_DUPS` | Don't record a command if it's identical to the previous one |
| `setopt APPEND_HISTORY` | Append to the history file instead of overwriting it on exit |
| `history \| grep <term>` | Search command history for a specific term |
| `Ctrl+R` | Interactive reverse history search |

### Prompt Customization

| Syntax | Description |
|---|---|
| `PROMPT='%n@%m %~ %# '` | Set the left-side prompt (`%n`=user, `%m`=host, `%~`=cwd, `%#`=`#`/`%` privilege indicator) |
| `RPROMPT='%T'` | Set a right-aligned prompt (e.g., showing the current time) |
| `autoload -Uz vcs_info` | Enable Git branch/status info in the prompt (used by most Zsh themes) |

### Oh My Zsh (Plugin Framework)

| Command / Config | Description |
|---|---|
| `omz update` | Update Oh My Zsh to the latest version |
| `plugins=(git docker zsh-autosuggestions)` in `.zshrc` | Enable specific plugins |
| `ZSH_THEME="robbyrussell"` in `.zshrc` | Set the active prompt theme |
| `omz theme list` | List all available built-in themes |

### Spelling Correction & Safety

| Setting | Description |
|---|---|
| `setopt CORRECT` | Suggest a correction for a mistyped command name |
| `setopt CORRECT_ALL` | Suggest corrections for mistyped arguments as well as command names |
| `setopt NO_CLOBBER` | Prevent `>` from accidentally overwriting an existing file (use `>!` to force) |

---

## Quick Diagnostic Commands

| Command | Purpose |
|---|---|
| `echo $ZSH_VERSION` | Confirm the current Zsh version |
| `echo $0` | Confirm which shell is actually interpreting the current script/session |
| `zsh -xv script.sh` | Run a script with verbose execution tracing (debug mode) |
| `which <command>` | Show which binary/alias/function will run for a given command name |
| `type <command>` | Show whether a name is a builtin, alias, function, or external binary |

## Prevention Tips

- Remember Zsh arrays are **1-indexed** — a common source of off-by-one bugs when porting scripts from Bash.
- Keep heavy logic (functions, loops, conditionals) in `.zshrc` only if truly needed for interactive use — excessive `.zshrc` complexity slows down every new terminal session's startup time.
- When writing portable scripts meant to run under multiple shells, avoid Zsh-only syntax (array indexing, `setopt`-dependent behavior) and target `sh`/Bash instead — see [Bourne Shell (sh) Cheat Sheet](./sh-cheatsheet.md) for the portable baseline.
