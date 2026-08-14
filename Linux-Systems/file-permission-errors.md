# File & Folder Permission Errors (chmod/chown)

## Problem / Context

Commands or applications fail with errors such as `Permission denied`, `Operation not permitted`, or scripts silently fail to read/write/execute a file — despite the file appearing to exist and the path being correct. This is one of the most common Linux issues, occurring in local development, deployment scripts, cron jobs, and Docker volume mounts alike.

Typical symptoms:
- `bash: ./script.sh: Permission denied` when trying to run a script.
- A web server (e.g., Nginx, Apache) returns `403 Forbidden` despite the file existing.
- `cp`/`mv`/`rm` fail with `Permission denied` even as the file's apparent owner.
- Cron jobs fail silently because the crontab user lacks access to a script or log path.
- Files created inside a Docker volume mount are owned by `root` and inaccessible to the host user.

## Root Cause

Permission errors stem from Linux's ownership and permission model, which controls access via three actor classes (**owner**, **group**, **others**) and three permission types (**read**, **write**, **execute**). Failures typically occur because of:

1. **Missing execute bit** — a script file lacks the `x` permission, so the kernel refuses to run it even if the content is valid.
2. **Ownership mismatch** — the file is owned by a different user/group (e.g., `root`) than the process attempting to access it.
3. **Restrictive permission mode** — the file/folder's permission bits (e.g., `600`, `700`) block access for the group or others.
4. **Directory traversal permission missing** — a file may have correct permissions, but a parent directory lacks the execute (`x`) bit needed to traverse into it.
5. **UID/GID mismatch in containers** — files created inside a Docker container (often as `root`, UID 0) are inaccessible or unmodifiable from the host user's account.
6. **SELinux/AppArmor context restrictions** (less common, mostly RHEL/CentOS) — permissions are correct at the Unix level, but a mandatory access control policy still blocks the operation.

## Resolution / Steps

### 1. Inspect Current Permissions and Ownership

```bash
ls -l path/to/file
```

Output breakdown:
```
-rwxr-xr--  1 alice  developers  1024  Aug 15 10:00  script.sh
 │└┬┘└┬┘└┬┘   │       │
 │ │  │  │    owner   group
 │ │  │  others permissions (r--)
 │ │  group permissions (r-x)
 │ owner permissions (rwx)
 file type (- = file, d = directory, l = symlink)
```

### 2. Fix "Permission Denied" When Executing a Script

Add the execute bit for the owner:

```bash
chmod u+x script.sh
```

Or, to allow execution for everyone:

```bash
chmod +x script.sh
```

Then run it:

```bash
./script.sh
```

### 3. Fix Ownership Mismatches

Check who currently owns the file:

```bash
ls -l path/to/file
# or
stat path/to/file
```

Change ownership to the correct user and group:

```bash
sudo chown alice:developers path/to/file
```

Apply recursively to a directory tree:

```bash
sudo chown -R alice:developers path/to/directory/
```

### 4. Adjust Permission Modes Precisely

Use symbolic mode for targeted changes (recommended — avoids accidentally over-granting access):

```bash
# Add write access for the group
chmod g+w file.txt

# Remove all access for others
chmod o-rwx file.txt
```

Use numeric (octal) mode for exact, absolute permission sets:

```bash
# rwx for owner, r-x for group, no access for others
chmod 750 file.txt

# rw- for owner and group, r-- for others (common for shared config files)
chmod 664 file.txt
```

| Octal | Permission | Common Use |
|---|---|---|
| `755` | `rwxr-xr-x` | Scripts, executables, directories |
| `644` | `rw-r--r--` | Regular files (configs, source code) |
| `700` | `rwx------` | Private scripts/directories (owner only) |
| `600` | `rw-------` | Sensitive files (SSH keys, credentials) |

### 5. Fix Directory Traversal Issues

If a file has correct permissions but is still inaccessible, check every parent directory in the path:

```bash
namei -l /full/path/to/file
```

This lists the permission of each path segment — identify which directory is missing the execute (`x`) bit and fix it:

```bash
chmod +x /full/path/to
```

### 6. Fix Docker Volume Permission Mismatches

Check the UID running inside the container:

```bash
docker exec -it <container_name> id
```

Align host directory ownership to match the container's UID/GID:

```bash
sudo chown -R 1000:1000 ./mounted-data
```

Alternatively, run the container as the host user to avoid root-owned output files:

```bash
docker run --user "$(id -u):$(id -g)" -v ./mounted-data:/app/data myimage
```

### 7. Check for SELinux/AppArmor Restrictions (RHEL/CentOS/Fedora)

If Unix permissions look correct but access is still denied:

```bash
# Check SELinux status
getenforce

# View SELinux context on a file
ls -Z path/to/file

# Restore the default context (common fix after moving/copying files)
sudo restorecon -v path/to/file
```

---

## Quick Diagnostic Commands

| Command | Purpose |
|---|---|
| `ls -l` | View permissions and ownership |
| `stat <file>` | Detailed metadata including UID/GID and access mode |
| `chmod +x <file>` | Grant execute permission |
| `chmod 755 <file>` | Set exact permission bits (octal) |
| `chown user:group <file>` | Change file owner and group |
| `chown -R user:group <dir>` | Recursively change ownership |
| `namei -l <path>` | Show permissions for every segment of a path |
| `id` | Show current user's UID/GID |
| `getenforce` | Check SELinux enforcement mode |

## Prevention Tips

- Never use `chmod 777` as a "quick fix" — it grants full read/write/execute to everyone and is a common security misconfiguration. Diagnose the actual permission gap instead.
- Set correct permissions at file creation time (e.g., via `umask` in shell profiles or deployment scripts) rather than patching after the fact.
- In Docker, prefer named volumes or explicit `--user` flags over relying on default root-owned bind mounts.
- For shared team directories, use the setgid bit (`chmod g+s directory/`) so new files automatically inherit the parent directory's group.
