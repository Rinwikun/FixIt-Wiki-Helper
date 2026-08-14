# Environment Variables & .env Not Loading

## Problem / Context

An application fails to read expected configuration values — API keys, database URLs, feature flags — despite a `.env` file existing in the project with seemingly correct entries. Symptoms include `undefined`/`null` values at runtime, fallback defaults being used unexpectedly, or the app crashing with errors like `Missing required environment variable`.

Typical trigger scenarios:
- Values work when run manually in the terminal but fail when run via a script, cron job, or process manager (PM2, systemd).
- Values work locally but not in CI/CD or in a Docker container.
- Changing `.env` has no effect until a full restart (or doesn't take effect even then).
- Some variables load correctly while others don't.

## Root Cause

`.env` loading failures generally fall into one of these categories:

1. **`.env` not loaded at all** — no library or mechanism (`dotenv`, Compose `env_file`, shell sourcing) is actually reading the file; the runtime relies purely on the OS environment.
2. **Wrong file location** — the `.env` file exists but isn't in the directory the process expects (commonly the current working directory, not the script's directory).
3. **Load order / precedence conflict** — OS-level environment variables, shell exports, or a process manager's own env config override `.env` values silently.
4. **Malformed syntax** — quoting, spacing, or line-ending issues (`KEY = value` with spaces, unescaped special characters, Windows CRLF line endings) cause silent parsing failures.
5. **Caching / stale process** — the app was started before `.env` was created or edited, and environment variables are only read once at process startup, not on every access.
6. **Docker-specific isolation** — `.env` is read by `docker compose` for variable substitution in the YAML file itself, which is a **separate mechanism** from `env_file:`/`environment:` passing variables into the container's runtime.
7. **`.gitignore`d and never actually created on target environment** — `.env` is (correctly) excluded from version control, but no `.env.example` → `.env` setup step was performed on the deployment target.

## Resolution / Steps

### 1. Confirm Whether Anything Is Loading `.env` At All

Node.js example — verify the loader is present and called early:

```javascript
// Must be one of the first lines executed, before any process.env access
require('dotenv').config();
```

Python example:

```python
from dotenv import load_dotenv
load_dotenv()  # Must run before os.getenv() calls
```

If no such call exists, `.env` is inert — it's just a text file the OS/runtime knows nothing about.

### 2. Verify the File's Location Relative to the Working Directory

`.env` loaders typically resolve relative to the **current working directory** (where the process was launched from), not the script's own directory.

```bash
# Confirm where you're actually running the command from
pwd
ls -la .env
```

If running from a different directory (e.g., a monorepo subfolder, or a cron job with a different working directory), specify the path explicitly:

```javascript
require('dotenv').config({ path: '/absolute/path/to/.env' });
```

### 3. Print Resolved Environment Values at Runtime

Add a temporary debug line immediately after the load call to confirm what was actually parsed:

```javascript
console.log('DATABASE_URL:', process.env.DATABASE_URL);
```

```bash
# Shell-level check
printenv | grep DATABASE_URL
```

If the value is `undefined`, the loader either isn't running or isn't finding the file — return to Step 1–2. If the value is present but wrong, proceed to Step 4 (precedence).

### 4. Check for Precedence Conflicts

Shell-exported or system-level environment variables typically **take priority** over `.env` file values in most `dotenv`-style libraries (they don't overwrite existing `process.env` keys by default).

```bash
# Check if the variable is already set at the shell/OS level
echo $DATABASE_URL
env | grep DATABASE_URL
```

If it's already set system-wide (e.g., in `~/.bashrc`, `~/.profile`, or a systemd unit's `Environment=`), that value wins over `.env` silently. Unset it for local testing:

```bash
unset DATABASE_URL
```

### 5. Fix Common Syntax Errors in `.env`

```dotenv
# ❌ Incorrect — spaces around "=" are not stripped by all parsers
DATABASE_URL = postgres://localhost/mydb

# ✅ Correct — no spaces around "="
DATABASE_URL=postgres://localhost/mydb

# ❌ Incorrect — unescaped special characters break parsing
API_KEY=abc$123!def

# ✅ Correct — wrap values containing special characters in quotes
API_KEY="abc$123!def"

# ❌ Incorrect — inline comments are not stripped by all parsers
PORT=3000 # default port

# ✅ Correct — comments on their own line
# default port
PORT=3000
```

Also check for **Windows CRLF line endings** if the file was edited on Windows and deployed to Linux:

```bash
# Detect CRLF endings
file .env

# Convert to Unix line endings
dos2unix .env
```

### 6. Restart the Process After Any `.env` Change

Environment variables are read once at process startup in most runtimes — editing `.env` while the app is running has no effect until restart.

```bash
# Node.js example with a process manager
pm2 restart app-name --update-env

# Plain restart
# (Ctrl+C, then re-run the start command)
```

### 7. Fix `.env` Loading in Docker Compose

Distinguish between the **two separate mechanisms**:

**a) `.env` for Compose file variable substitution** (used to fill in `${VARIABLE}` placeholders inside `docker-compose.yml` itself):

```yaml
# docker-compose.yml
services:
  app:
    image: "myapp:${TAG}"   # ${TAG} is substituted from .env in the project root
```

This `.env` **must** be in the same directory as `docker-compose.yml` — Compose does not search elsewhere automatically.

**b) `env_file:` for passing variables into the container's runtime:**

```yaml
services:
  app:
    env_file:
      - .env.production   # explicitly loaded into the container's environment
```

Verify what the container actually receives:

```bash
docker exec -it <container_name> env
```

### 8. Confirm `.env` Actually Exists on the Target Environment

Since `.env` is (correctly) excluded via `.gitignore`, deployment targets and fresh clones won't have it unless explicitly created:

```bash
# Check if .env exists at all
ls -la .env

# If missing, copy from the example template and fill in real values
cp .env.example .env
```

---

## Quick Diagnostic Commands

| Command | Purpose |
|---|---|
| `printenv \| grep <VAR>` | Check if a variable is set at the shell/OS level |
| `env` | List all environment variables in the current shell |
| `pwd` | Confirm current working directory (relevant to relative `.env` paths) |
| `docker exec -it <container> env` | View environment variables inside a running container |
| `docker compose config` | Preview resolved Compose config with `.env` substitution applied |
| `file .env` | Detect line-ending issues (CRLF vs LF) |
| `dos2unix .env` | Convert Windows line endings to Unix |

## Prevention Tips

- Always commit a `.env.example` with placeholder values so the required variables are documented and discoverable.
- Load environment configuration as the very first action in the application's entry point, before any other imports that might read `process.env`.
- Avoid mixing shell-exported variables and `.env` files for the same keys — pick one source of truth per environment to prevent silent precedence conflicts.
- In CI/CD, prefer the pipeline's native secrets/variables mechanism over shipping a `.env` file, and confirm the pipeline explicitly maps those secrets into the runtime environment.
