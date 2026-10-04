# Historical Shells (Reference Only)

## Problem / Context

The shells listed here are no longer in active production use but represent important lineage points in shell history — understanding them clarifies *why* modern shells (`sh`, `bash`, `ksh`, `csh`) are designed the way they are. This document is a brief historical reference, not an operational cheat sheet — none of these should be used for new scripting work.

## Root Cause

Not applicable — this is a historical reference, not a troubleshooting document.

## Resolution / Steps

### Reference Table

| Shell | Era | Significance | Why It's Obsolete |
|---|---|---|---|
| **Thompson Shell** | 1971 | The first Unix shell, written by Ken Thompson. Introduced the basic concept of a command interpreter with I/O redirection. | Extremely primitive by modern standards — no scripting constructs (no `if`, loops, or variables as understood today). Fully superseded by the PWB/Mashey Shell and later the Bourne Shell. |
| **PWB/Mashey Shell** | 1975 | Developed by John Mashey for the Programmer's Workbench Unix. Added shell variables, loops (`for`, `while`), and the foundation of shell scripting as a real programming paradigm. | Directly superseded by the Bourne Shell (1977), which became the long-term Unix standard and incorporated/refined its scripting ideas. |
| **pdksh (Public Domain Korn Shell)** | 1980s–2000s | A free, open-source clone of AT&T's proprietary `ksh88`, widely used on early open-source Unix-likes (OpenBSD, early Linux) before `ksh93` was open-sourced. | Development effectively stopped in the mid-2000s. Functionally replaced by **mksh** (MirBSD Korn Shell), a actively maintained continuation — see [mksh cheat sheet](./mksh-cheatsheet.md). |
| **Heirloom Bourne Shell** | 2000s (port) | A modern port/preservation of the original AT&T Bourne Shell source code, maintained as part of the "Heirloom Project" for historical and portability research. | Niche preservation project — not intended or recommended for production scripting; `dash` and POSIX `sh` serve that role today. |
| **COMMAND.COM** | 1981–2000s (MS-DOS, Windows 9x) | The original command interpreter for MS-DOS and the 9x line of Windows (95/98/ME). Batch file (`.bat`) scripting originates here. | Fully replaced by `cmd.exe` starting with Windows NT, and modern systems no longer run MS-DOS or Windows 9x. |
| **AmigaShell** | 1985 onward (AmigaOS) | The command-line shell for AmigaOS, notable for its own scripting dialect (ARexx integration) distinct from Unix shell conventions. | Tied to AmigaOS, which has negligible modern market presence outside hobbyist/retro-computing communities. |
| **DCL (DIGITAL Command Language)** | 1970s onward (OpenVMS) | The native command shell/scripting language for DEC's OpenVMS operating system, still present on VMS systems used in some legacy enterprise/government infrastructure. | Not obsolete in the sense of being unused everywhere — OpenVMS systems still run in specific legacy enterprise niches — but outside that narrow context, it has no relevance to mainstream Unix/Windows/Linux work. |

---

## Why This Lineage Matters

- The **Thompson Shell → PWB/Mashey Shell → Bourne Shell** progression shows how scripting constructs (`if`, `for`, variables) were incrementally invented, not present from Unix's very first shell.
- **pdksh → mksh** demonstrates a common open-source pattern: a stalled community fork gets succeeded by an actively maintained continuation rather than disappearing outright.
- **COMMAND.COM → cmd.exe** mirrors the same kind of generational replacement on the Windows side — see [CMD cheat sheet](./cmd-cheatsheet.md) for the modern equivalent.

## Prevention Tips

- If working with legacy OpenVMS infrastructure specifically, DCL is still genuinely relevant — this is the one entry in this list that isn't purely historical trivia.
- For everything else here, there is no practical reason to target these shells for new work — use their modern, actively maintained equivalents linked above instead.
