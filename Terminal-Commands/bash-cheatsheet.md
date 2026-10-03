# Bash (Bourne-Again Shell) Cheat Sheet

## Problem / Context

Bash remains the default shell on most Linux distributions. Daily Bash work requires quick recall of commonly used commands for navigation, file management, process inspection, and system diagnostics. This cheat sheet consolidates the most frequently used commands with brief explanations, scoped to Bash syntax (prompt style: `user@host:~$`).

## Root Cause

Not applicable — this is a reference guide rather than a troubleshooting document.

## Resolution / Steps

### Navigation & File Management

| Command | Description |
|---|---|
| `pwd` | Print current working directory |
| `ls -la` | List all files (including hidden) with details |
| `cd <path>` | Change directory |
| `mkdir -p <path>` | Create a directory, including parent directories if needed |
| `rm -rf <path>` | Remove a file or directory recursively and forcefully |
| `cp -r <src> <dest>` | Copy files/directories recursively |
| `mv <src> <dest>` | Move or rename a file/directory |
| `find <path> -name "<pattern>"` | Search for files matching a pattern |
| `touch <file>` | Create an empty file or update its timestamp |

### Viewing & Editing Files

| Command | Description |
|---|---|
| `cat <file>` | Print entire file content to terminal |
| `less <file>` | View file content page-by-page (scrollable) |
| `head -n 20 <file>` | Show the first 20 lines of a file |
| `tail -f <file>` | Follow a file's new content in real time (e.g., live logs) |
| `nano <file>` / `vim <file>` | Open a file in a terminal text editor |
| `grep -rn "<pattern>" <path>` | Search recursively for a text pattern, showing line numbers |

### Permissions & Ownership

| Command | Description |
|---|---|
| `chmod +x <file>` | Add execute permission |
| `chmod 755 <file>` | Set exact permission bits (owner rwx, group/others r-x) |
| `chown user:group <file>` | Change file owner and group |
| `sudo <command>` | Run a command with elevated (root) privileges |

### Process & System Monitoring

| Command | Description |
|---|---|
| `ps aux` | List all running processes |
| `top` / `htop` | Interactive real-time process/resource monitor |
| `kill <PID>` | Terminate a process by its process ID |
| `kill -9 <PID>` | Force-terminate a process (SIGKILL) |
| `df -h` | Show disk space usage in human-readable format |
| `du -sh <path>` | Show total size of a directory |
| `free -h` | Show RAM and swap usage |

### Networking

| Command | Description |
|---|---|
| `ping <host>` | Test connectivity to a host |
| `curl <url>` | Send an HTTP request and print the response |
| `netstat -tulpn` | Show listening ports and associated processes |
| `ssh user@host` | Connect to a remote machine via SSH |
| `scp <file> user@host:<path>` | Copy a file to/from a remote machine over SSH |

### Package Management (Debian/Ubuntu)

| Command | Description |
|---|---|
| `sudo apt update` | Refresh the local package index |
| `sudo apt install <package>` | Install a package |
| `sudo apt remove <package>` | Remove a package |
| `sudo apt upgrade` | Upgrade all installed packages |

### Service Management (systemd)

| Command | Description |
|---|---|
| `systemctl status <service>` | Check a service's current status |
| `sudo systemctl start <service>` | Start a service |
| `sudo systemctl restart <service>` | Restart a service |
| `sudo systemctl enable <service>` | Enable a service to start on boot |

---

## Quick Diagnostic Commands

| Command | Purpose |
|---|---|
| `history` | View previously run commands |
| `man <command>` | Open the manual page for a command |
| `which <command>` | Show the full path of an executable |
| `uname -a` | Display kernel and system information |

## Prevention Tips

- Double-check the target path before running `rm -rf` — it does not ask for confirmation and is irreversible.
- Prefer `kill` (graceful SIGTERM) over `kill -9` unless the process is unresponsive, to allow proper cleanup.
- Use `man <command>` or `<command> --help` when unsure of flags rather than guessing.
