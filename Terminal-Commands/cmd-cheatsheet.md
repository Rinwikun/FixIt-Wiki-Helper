# CMD (Command Prompt) Cheat Sheet

## Problem / Context

Command Prompt (`cmd.exe`) remains widely used on Windows for scripting, legacy tooling, and quick system operations. This cheat sheet consolidates commonly used `cmd` commands with brief explanations, scoped strictly to classic Command Prompt syntax (prompt style: `C:\Users\[Type your laptop brand name or your name]>`).

## Root Cause

Not applicable — this is a reference guide rather than a troubleshooting document.

## Resolution / Steps

### Navigation & Directory Management

| Command | Description |
|---|---|
| `cd <path>` | Change directory |
| `cd ..` | Move up one directory level |
| `cd /d D:\path` | Change drive and directory in one step |
| `dir` | List files and folders in the current directory |
| `dir /a` | List all files, including hidden/system files |
| `dir /s` | List files recursively through subdirectories |
| `tree` | Display directory structure as a tree |
| `mkdir <path>` | Create a new directory |
| `rmdir /s /q <path>` | Delete a directory and all its contents, no confirmation prompt |

### File Operations

| Command | Description |
|---|---|
| `copy <src> <dest>` | Copy a file |
| `xcopy <src> <dest> /E /H /C /I` | Copy directories including subfolders, hidden files, continue on error |
| `robocopy <src> <dest> /E` | Robust file copy with resume support (preferred over `xcopy` for large/critical transfers) |
| `move <src> <dest>` | Move or rename a file |
| `del <file>` | Delete a file |
| `del /q /f <file>` | Force-delete a file without confirmation |
| `ren <old> <new>` | Rename a file |
| `type <file>` | Print file content to the terminal |
| `fc <file1> <file2>` | Compare two files and show differences |

### Text Search & Processing

| Command | Description |
|---|---|
| `findstr "<pattern>" <file>` | Search for a text pattern inside a file |
| `findstr /s /i "<pattern>" *.txt` | Case-insensitive recursive search across matching files |
| `sort <file>` | Sort the lines of a file alphabetically |
| `more <file>` | View file content one screen at a time |

### System Information & Diagnostics

| Command | Description |
|---|---|
| `systeminfo` | Display detailed OS, hardware, and patch information |
| `ver` | Show the Windows version |
| `hostname` | Display the computer's hostname |
| `whoami` | Show the currently logged-in user |
| `whoami /priv` | Show the current user's privilege list |
| `echo %USERNAME%` | Print the current username via environment variable |
| `set` | List all current environment variables |
| `setx <VAR> "<value>"` | Permanently set an environment variable for the current user |

### Process Management

| Command | Description |
|---|---|
| `tasklist` | List all running processes |
| `tasklist /fi "imagename eq chrome.exe"` | Filter running processes by name |
| `taskkill /PID <pid> /F` | Force-terminate a process by its Process ID |
| `taskkill /IM <name> /F` | Force-terminate a process by image name |
| `start <program>` | Launch a program in a new window |

### Disk & Storage

| Command | Description |
|---|---|
| `chkdsk C: /f` | Check a drive for errors and fix them |
| `diskpart` | Open the interactive disk partitioning utility |
| `wmic logicaldisk get size,freespace,caption` | Show disk size and free space per drive letter |
| `format <drive>:` | Format a drive (⚠ destructive — erases all data) |

### Networking

| Command | Description |
|---|---|
| `ipconfig /all` | Show full network adapter configuration |
| `ipconfig /release` then `ipconfig /renew` | Release and renew a DHCP-assigned IP address |
| `ipconfig /flushdns` | Flush the local DNS resolver cache |
| `ping <host>` | Test connectivity to a host |
| `tracert <host>` | Trace the network route to a host |
| `nslookup <domain>` | Query DNS resolution for a domain |
| `netstat -ano` | Show active connections and listening ports with owning PIDs |
| `netsh wlan show profiles` | List saved Wi-Fi profiles |
| `netsh wlan show profile name="<SSID>" key=clear` | Reveal a saved Wi-Fi password |

### Service Management

| Command | Description |
|---|---|
| `sc query <service>` | Check a Windows service's current status |
| `net start <service>` | Start a service |
| `net stop <service>` | Stop a service |
| `net start` (no argument) | List all currently running services |

### User & Permission Management

| Command | Description |
|---|---|
| `net user` | List all local user accounts |
| `net user <username>` | Show details for a specific user account |
| `net user <username> <password>` | Change a local user's password |
| `net localgroup administrators` | List members of the local Administrators group |
| `icacls <path>` | View or modify file/folder permissions (ACLs) |

### Batch Scripting Basics

| Syntax | Description |
|---|---|
| `@echo off` | Suppress command echoing in a batch script (first line convention) |
| `%1`, `%2`, ... | Reference positional arguments passed to a batch script |
| `if exist <file> ( ... )` | Conditional check for file existence |
| `for %%f in (*.txt) do echo %%f` | Loop over files matching a pattern |
| `pause` | Halt script execution until a key is pressed |
| `exit /b 0` | Exit a batch script with a specific exit code |

---

## Quick Diagnostic Commands

| Command | Purpose |
|---|---|
| `doskey /history` | View previously run commands in the session |
| `help` | List all available built-in commands |
| `help <command>` | Show usage and syntax for a specific command |
| `where <command>` | Show the full path of an executable in PATH |
| `cls` | Clear the terminal screen |

## Prevention Tips

- `rmdir /s /q`, `del /q /f`, and `format` are irreversible — double-check the target path/drive before executing.
- Run Command Prompt as Administrator when a command fails with "Access is denied" (`net start`, `chkdsk`, `diskpart`, etc. often require elevation).
- Prefer `robocopy` over `xcopy`/`copy` for large or critical file transfers — it supports resuming interrupted copies and retry logic.
