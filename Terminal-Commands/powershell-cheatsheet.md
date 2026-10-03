# PowerShell Terminal Cheat Sheet

## Problem / Context

PowerShell is the modern default shell on Windows, built around structured object pipelines rather than plain text. This cheat sheet consolidates commonly used PowerShell cmdlets with brief explanations, scoped strictly to PowerShell syntax (prompt style: `PS C:\Users\LOQ>`).

## Root Cause

Not applicable — this is a reference guide rather than a troubleshooting document.

## Resolution / Steps

### Navigation & Directory Management

| Command | Description |
|---|---|
| `Set-Location <path>` / `cd <path>` | Change directory |
| `Get-Location` / `pwd` | Print current working directory |
| `Get-ChildItem` / `ls` / `dir` | List files and folders |
| `Get-ChildItem -Recurse` | List files recursively through subdirectories |
| `Get-ChildItem -Force` | Include hidden/system files in the listing |
| `New-Item -ItemType Directory -Path <path>` | Create a new directory |
| `Remove-Item <path> -Recurse -Force` | Delete a directory and all its contents, no confirmation prompt |

### File Operations

| Command | Description |
|---|---|
| `Copy-Item <src> <dest>` | Copy a file |
| `Copy-Item <src> <dest> -Recurse` | Copy a directory and its contents |
| `Move-Item <src> <dest>` | Move or rename a file |
| `Remove-Item <file>` | Delete a file |
| `Rename-Item <old> <new>` | Rename a file |
| `Get-Content <file>` | Print file content to the terminal |
| `Get-Content <file> -Tail 20 -Wait` | Follow a file's new content in real time (like `tail -f`) |
| `Set-Content <file> "<text>"` | Overwrite a file with the given text |
| `Add-Content <file> "<text>"` | Append text to a file |
| `Compare-Object (Get-Content f1) (Get-Content f2)` | Compare two files line-by-line |

### Text Search & Processing

| Command | Description |
|---|---|
| `Select-String "<pattern>" <file>` | Search for a text pattern inside a file |
| `Get-ChildItem -Recurse \| Select-String "<pattern>"` | Recursively search a text pattern across files |
| `Sort-Object` | Sort pipeline objects by a property |
| `Where-Object { $_.Property -eq "value" }` | Filter pipeline objects by condition |
| `Select-Object -First 10` | Return only the first N objects from the pipeline |
| `ConvertTo-Json` / `ConvertFrom-Json` | Convert objects to/from JSON format |

### System Information & Diagnostics

| Command | Description |
|---|---|
| `Get-ComputerInfo` | Display detailed OS, hardware, and system information |
| `$PSVersionTable` | Show the current PowerShell version and build info |
| `hostname` | Display the computer's hostname |
| `whoami` | Show the currently logged-in user |
| `whoami /groups` | Show the current user's group memberships |
| `Get-ChildItem Env:` | List all environment variables |
| `[Environment]::SetEnvironmentVariable("VAR","value","User")` | Permanently set a user-level environment variable |

### Process Management

| Command | Description |
|---|---|
| `Get-Process` | List all running processes |
| `Get-Process -Name chrome` | Filter running processes by name |
| `Stop-Process -Id <pid> -Force` | Force-terminate a process by its Process ID |
| `Stop-Process -Name <name> -Force` | Force-terminate a process by name |
| `Start-Process <program>` | Launch a program |
| `Start-Process <program> -Verb RunAs` | Launch a program with elevated (Administrator) privileges |

### Disk & Storage

| Command | Description |
|---|---|
| `Get-Volume` | Show disk volumes with size and free space |
| `Get-PSDrive` | List all mounted drives, including non-filesystem drives |
| `Get-Disk` | List physical disks attached to the system |
| `Repair-Volume -DriveLetter C` | Check and repair a drive for errors |

### Networking

| Command | Description |
|---|---|
| `Get-NetIPConfiguration` | Show network adapter configuration |
| `Test-Connection <host>` | Test connectivity to a host (PowerShell equivalent of `ping`) |
| `Test-NetConnection <host> -Port 443` | Test TCP connectivity to a specific port |
| `Clear-DnsClientCache` | Flush the local DNS resolver cache |
| `Resolve-DnsName <domain>` | Query DNS resolution for a domain |
| `Get-NetTCPConnection` | Show active TCP connections and listening ports |
| `Get-NetAdapter` | List network adapters and their status |

### Service Management

| Command | Description |
|---|---|
| `Get-Service` | List all services and their status |
| `Get-Service <name>` | Check a specific service's current status |
| `Start-Service <name>` | Start a service |
| `Stop-Service <name>` | Stop a service |
| `Restart-Service <name>` | Restart a service |

### User & Permission Management

| Command | Description |
|---|---|
| `Get-LocalUser` | List all local user accounts |
| `Get-LocalGroupMember -Group "Administrators"` | List members of the local Administrators group |
| `Get-Acl <path>` | View file/folder permissions (ACLs) |
| `Set-Acl <path> <aclObject>` | Apply permissions (ACLs) to a file/folder |

### Package Management (winget)

| Command | Description |
|---|---|
| `winget search <name>` | Search for an available package |
| `winget install <name>` | Install a package |
| `winget uninstall <name>` | Uninstall a package |
| `winget upgrade --all` | Upgrade all installed packages |

### Module & Environment Management

| Command | Description |
|---|---|
| `Get-Module -ListAvailable` | List all installed PowerShell modules |
| `Install-Module <name>` | Install a module from the PowerShell Gallery |
| `Import-Module <name>` | Load a module into the current session |
| `Get-ExecutionPolicy` | Check the current script execution policy |
| `Set-ExecutionPolicy RemoteSigned -Scope CurrentUser` | Allow locally-written scripts to run (common dev setup) |

---

## Quick Diagnostic Commands

| Command | Purpose |
|---|---|
| `Get-History` | View previously run commands in the session |
| `Get-Help <command>` | Show usage and syntax for a cmdlet |
| `Get-Help <command> -Examples` | Show usage examples for a cmdlet |
| `Get-Command <name>` | Find a cmdlet by name or partial match |
| `Get-Alias` | List all built-in command aliases (e.g., `ls` → `Get-ChildItem`) |
| `cls` / `Clear-Host` | Clear the terminal screen |

## Prevention Tips

- `Remove-Item -Recurse -Force` is irreversible — double-check the target path before executing.
- Run PowerShell as Administrator (or use `Start-Process -Verb RunAs`) when a cmdlet fails with an access-denied error.
- Prefer PowerShell cmdlets over invoking legacy CMD commands for scripting — cmdlets return structured objects (not plain text), making filtering and automation far more reliable (`Where-Object`, `Select-Object`, etc.).
- Be cautious when changing `Set-ExecutionPolicy` — `Unrestricted` removes script-safety checks; `RemoteSigned` is the safer common default for development machines.
