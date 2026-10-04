# Niche & Modern Experimental Shells

## Problem / Context

Beyond the mainstream shells covered elsewhere in this collection, a long tail of niche, academic, and experimental shells exists — each exploring a different idea about what a shell could be (Python-powered, Scheme-powered, Rust-powered, Lua-configurable, strict POSIX-only, etc.). Most aren't practical day-to-day choices for troubleshooting work, but knowing what they are helps when encountering them in the wild (a Redox OS system, an Emacs-centric workflow, a Debian portability-testing pipeline). This document is a survey reference, not a full operational cheat sheet.

## Root Cause

Not applicable — this is a survey/reference document, not a troubleshooting document.

## Resolution / Steps

### Survey Table

| Shell | Language / Platform | Notable Feature | Typical Use Case |
|---|---|---|---|
| **Elvish** | Go | Structured-data pipelines (similar concept to Nushell/PowerShell) with its own expressive scripting language | Users wanting a modern, structured-pipeline shell as a daily driver |
| **Xonsh** | Python (superset) | Mix Python and shell syntax directly in the same line — `ls` and `for x in range(10):` both work | Python-heavy workflows wanting to avoid context-switching between shell and Python scripts |
| **Oils (OSH/YSH)** | C++/Python (self-hosted) | **OSH** is a drop-in Bash-compatible runtime; **YSH** is a new, safer scripting language built on the same engine | Teams wanting Bash compatibility today with a migration path to a less error-prone scripting language |
| **Murex** | Go | "Typed pipelines" — data retains type information (string, JSON, CSV) as it flows between commands | Scripting that benefits from type safety without adopting a full structured-data shell like Nushell |
| **Ion** | Rust | Default shell of Redox OS (a Rust-written OS); designed for speed and safety | Exclusively relevant on Redox OS systems |
| **Hilbish** | Go, Lua-configurable | Entire shell configuration/scripting done in Lua instead of a shell-specific language | Users who already know Lua and want that as their shell's scripting language |
| **rc** | C | Plan 9's shell — simple, consistent C-like syntax, deliberately avoids Bourne shell's syntactic quirks | Plan 9 systems, or Unix users specifically seeking rc's cleaner syntax philosophy |
| **es** | C | "Extensible shell" — influenced by both `rc` and Scheme, treats functions as first-class values | Academic/research interest in shell language design |
| **scsh (Scheme Shell)** | Scheme | Uses full Scheme as the scripting language, with Unix system-call bindings | Scheme/Lisp programmers wanting a shell that's "really" a general-purpose language |
| **Eshell** | Emacs Lisp | A shell implemented *inside* Emacs itself — no external process; Lisp and shell commands can mix | Users who live inside Emacs and want a shell without leaving the editor |
| **yash** | C (POSIX-focused) | Extremely strict POSIX compliance, plus Unicode-aware string handling | Validating strict POSIX script behavior, non-ASCII text processing edge cases |
| **posh (Policy-Compliant Ordinary Shell)** | C (minimal) | Deliberately minimal — used by Debian specifically to catch non-portable `#!/bin/sh` scripts during package QA | Portability testing in Debian packaging pipelines (not intended as a daily-use shell) |
| **ksh93u+m** | C | Actively maintained community fork of `ksh93`, continuing development after AT&T's original ceased | Teams wanting modern `ksh93` features with ongoing security/bug fixes |
| **oksh** | C | OpenBSD's own port/trim of the public-domain `ksh`, tuned for OpenBSD's minimalism philosophy | OpenBSD systems, or users wanting a smaller `ksh` than `mksh`/`ksh93` |
| **Take Command / 4NT** | Proprietary (JPSoft, Windows) | Commercial `cmd.exe` replacement with a vastly extended command set, tabbed interface, and scripting enhancements | Windows power users/enterprises wanting a richer alternative to CMD without switching to PowerShell |

### A Closer Look: The Two Most Actively Relevant Today

**Xonsh** — mixing Python and shell on the same line:

```xonsh
# Shell command
ls -la

# Python, directly in the same session
for i in range(3):
    print(i)

# Mixing both: capture shell output into a Python variable
files = $(ls).split()
print(len(files))
```

**Elvish** — structured pipelines, similar philosophy to Nushell:

```elvish
# ls output is structured; pipe into further processing
ls | each {|f| echo $f}

# Variable assignment
var x = 5
echo $x
```

See [Nushell Cheat Sheet](./nushell-cheatsheet.md) for a deeper look at the structured-pipeline concept Elvish and Murex both share.

---

## Why These Aren't Covered as Full Cheat Sheets

Unlike `bash`, `zsh`, `fish`, or `ksh`, these shells are rarely the shell a troubleshooting scenario will actually involve — they're either tied to a specific niche platform (Redox's Ion, Debian QA's posh), an editor-embedded context (Eshell), or genuinely experimental/low-adoption choices (es, scsh, rc, Hilbish, Murex). A full step-by-step cheat sheet for each would add repository weight without matching real-world lookup frequency.

## Prevention Tips

- If you encounter one of these shells unexpectedly (e.g., a script with `#!/usr/bin/env xonsh` or `#!/usr/bin/env elvish`), don't assume Bash/POSIX syntax applies — check that shell's own documentation rather than guessing from this table alone.
- `posh` specifically exists to **break** scripts that rely on bashisms — if a Debian package build fails under `posh` but works under `bash`, that's a portability bug in the script, not a `posh` bug. See [Dash Cheat Sheet](./dash-cheatsheet.md) for the same class of issue.
- For genuinely adopting a modern structured-pipeline shell day-to-day, Nushell (already covered in this collection) has notably more mainstream traction and documentation than Elvish or Murex as of this writing — worth trying first before the more niche alternatives.
