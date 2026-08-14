# Common Containerization & Orchestration Issues (Docker & Docker Compose)

## Problem / Context

Docker and Docker Compose introduce a distinct set of recurring failure modes that differ from traditional bare-metal or VM troubleshooting — networking between isolated containers, volume permission mismatches, build context errors, and resource constraints imposed by the container runtime. This document consolidates the most frequently encountered issues by operational category, along with root causes and resolution steps.

## Root Cause

Not applicable at the document level — see the **Root Cause** subsection under each category below, as causes vary by issue type.

## Resolution / Steps

### 1. Port Binding & Networking Issues

**Port Conflict**
- **Root Cause:** The host port defined in `ports:` (e.g., `8080:80`) is already bound by another process or container.
- **Resolution:**
  ```bash
  # Identify what is using the port on the host
  lsof -i :8080
  # or
  sudo netstat -tulpn | grep 8080

  # Fix: change the host-side port mapping
  # docker-compose.yml
  ports:
    - "8081:80"
  ```

**Inter-Container Communication Failure**
- **Root Cause:** Containers reference each other by IP or `localhost` instead of the Compose **service name**, or they sit on different Docker networks and cannot resolve one another.
- **Resolution:**
  ```yaml
  # Use the service name as the hostname, not localhost or a hardcoded IP
  environment:
    - DATABASE_HOST=db   # "db" is the service name defined in docker-compose.yml
  ```
  Verify both services share the same network:
  ```bash
  docker network inspect <network_name>
  ```

**Localhost Resolution Misuse**
- **Root Cause:** `localhost` inside a container refers to the container's own network namespace, not the host or sibling containers.
- **Resolution:** Always use the Compose service name for inter-service calls. Reserve `localhost` only for processes within the *same* container.

---

### 2. Volume & Data Persistence Issues

**Data Loss on Restart/Remove**
- **Root Cause:** No volume or bind mount was defined, so data was written to the container's writable layer, which is destroyed with `docker rm`.
- **Resolution:**
  ```yaml
  volumes:
    - db-data:/var/lib/mysql   # named volume, persists across container lifecycle

  volumes:
    db-data:
  ```

**Permission Denied on Mounted Volumes**
- **Root Cause:** UID/GID mismatch between the host user and the user running inside the container (common on Linux hosts).
- **Resolution:**
  ```bash
  # Check the container's running user
  docker exec -it <container_name> id

  # Align host directory ownership with container UID/GID
  sudo chown -R 1000:1000 ./data

  # Or run the container with a matching UID
  docker run --user "$(id -u):$(id -g)" ...
  ```

---

### 3. Build & Dependency Failures

**Build Context Error (COPY/ADD failure)**
- **Root Cause:** `COPY`/`ADD` paths are resolved relative to the **build context**, not the `Dockerfile`'s location — a common source of "file not found" errors.
- **Resolution:** Confirm the build context in `docker-compose.yml` or the `docker build` command includes the required files:
  ```bash
  docker build -t myapp -f ./docker/Dockerfile .
  ```

**Architecture Mismatch**
- **Root Cause:** An image built for one CPU architecture (e.g., `linux/amd64`) is run on an incompatible host (e.g., Apple Silicon / ARM).
- **Resolution:**
  ```bash
  # Build a multi-architecture image
  docker buildx build --platform linux/amd64,linux/arm64 -t myapp:latest .

  # Or explicitly pull/run the matching platform
  docker run --platform linux/amd64 myapp:latest
  ```

**Dependency Caching Issues**
- **Root Cause:** Stale Docker layer cache causes outdated dependencies to persist across builds.
- **Resolution:**
  ```bash
  docker build --no-cache -t myapp .
  ```

---

### 4. Resource & Runtime Limits

**Out of Memory (OOM Killed)**
- **Root Cause:** Container memory usage exceeds the limit imposed by Docker or the host, triggering the kernel OOM killer.
- **Resolution:**
  ```bash
  # Inspect exit code / OOM status
  docker inspect <container_name> --format='{{.State.OOMKilled}}'
  ```
  ```yaml
  # Set explicit, realistic memory limits
  deploy:
    resources:
      limits:
        memory: 512M
  ```

**High CPU / Resource Bottleneck**
- **Root Cause:** No CPU limits or reservations are configured, allowing a single container to starve others on the same host.
- **Resolution:**
  ```yaml
  deploy:
    resources:
      limits:
        cpus: "1.0"
      reservations:
        cpus: "0.5"
  ```
  Monitor live usage:
  ```bash
  docker stats
  ```

---

### 5. Docker Compose Configuration Errors

**YAML Syntax Error**
- **Root Cause:** Incorrect indentation, tabs instead of spaces, or malformed key-value structure in `docker-compose.yml`.
- **Resolution:**
  ```bash
  docker compose config
  ```
  This validates and prints the fully resolved configuration, surfacing syntax errors before `up`.

**Version Compatibility**
- **Root Cause:** The `version:` field in `docker-compose.yml` is incompatible with the installed Docker Compose CLI version.
- **Resolution:**
  ```bash
  docker compose version
  ```
  Note: Compose Specification (v2+) has deprecated the top-level `version:` key entirely — omitting it is now the recommended approach for current Compose CLI versions.

---

### 6. Additional Common Issues

**Orphaned Containers, Volumes & Images**
- **Root Cause:** Stopped containers, dangling images, and unused volumes accumulate over time, consuming disk space.
- **Resolution:**
  ```bash
  # Preview what would be removed
  docker system df

  # Clean up unused data (containers, networks, dangling images)
  docker system prune

  # Include unused volumes (destructive — verify first)
  docker system prune --volumes
  ```

**Log / Disk Bloat**
- **Root Cause:** The default `json-file` logging driver has no size limit by default, allowing container logs to grow unbounded and fill host disk.
- **Resolution:**
  ```yaml
  logging:
    driver: "json-file"
    options:
      max-size: "10m"
      max-file: "3"
  ```

**Environment Variables Not Loading**
- **Root Cause:** `.env` file is not in the Compose project root, or confusion between `environment:` (explicit values) and `env_file:` (external file reference) precedence.
- **Resolution:**
  ```bash
  # Verify resolved environment values
  docker compose config
  ```
  Ensure `.env` sits alongside `docker-compose.yml` in the project root — Compose loads it automatically only from that location.

**Restart Policy Misconfiguration**
- **Root Cause:** No `restart:` policy defined, so containers do not recover automatically after a crash or host reboot.
- **Resolution:**
  ```yaml
  restart: unless-stopped
  ```

**Healthcheck Misconfiguration (Startup Race Conditions)**
- **Root Cause:** Compose's `depends_on` only waits for a container to *start*, not for the application inside it to be *ready*, causing dependent services to fail on first connection attempt.
- **Resolution:**
  ```yaml
  services:
    db:
      healthcheck:
        test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
        interval: 5s
        timeout: 3s
        retries: 5

    app:
      depends_on:
        db:
          condition: service_healthy
  ```

---

## Quick Diagnostic Commands

| Command | Purpose |
|---|---|
| `docker ps -a` | List all containers, including stopped ones |
| `docker logs <container>` | View container logs |
| `docker inspect <container>` | Full metadata: mounts, network, env, state |
| `docker compose config` | Validate and preview resolved Compose config |
| `docker system df` | Disk usage summary (images, containers, volumes) |
| `docker stats` | Live CPU/memory usage per container |
| `docker network inspect <network>` | Verify which containers share a network |
