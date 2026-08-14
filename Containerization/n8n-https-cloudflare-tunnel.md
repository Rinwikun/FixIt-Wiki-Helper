# Enabling HTTPS for n8n via Cloudflare Tunnel

## Problem / Context

By default, a self-hosted n8n instance (via Docker) runs over plain **HTTP** on `localhost:5678`, with no built-in TLS/HTTPS termination. This is fine for purely local use, but becomes a blocker when:

- n8n needs to receive **webhooks from external services** (e.g., Stripe, GitHub, Telegram), which typically require a public **HTTPS** endpoint.
- The instance needs to be reachable from outside the local network without manually configuring port forwarding, a reverse proxy, or a TLS certificate.

Running `docker run -p 5678:5678 n8nio/n8n` alone only exposes the container over HTTP on the local machine/network — it does not provide a public URL or HTTPS.

## Root Cause

n8n has no awareness of how it is being accessed externally unless explicitly told. Two separate problems need solving:

1. **No public HTTPS endpoint exists** — the container is only bound to `localhost`/LAN; nothing exposes it to the internet or terminates TLS.
2. **n8n generates webhook URLs based on its own configured URL** — if n8n isn't told its externally-reachable HTTPS address (via `WEBHOOK_URL` / `N8N_WEBHOOK_URL` / `N8N_PROTOCOL`), it will generate webhook URLs pointing to `http://localhost:5678/...`, which are unreachable by any external service.

**Cloudflare Tunnel** (`cloudflared`) solves the first problem by creating an outbound-only tunnel from the local machine to Cloudflare's edge, which issues a public HTTPS URL (`*.trycloudflare.com` for quick tunnels) — without opening any inbound ports or configuring a certificate manually. The second problem is solved by explicitly passing that tunnel URL into n8n's environment variables at startup.

**Note:** Most n8n HTTPS tutorials rely on a **paid** domain plus a reverse proxy (Nginx/Caddy) and a purchased/managed TLS certificate. The approach below uses Cloudflare's **quick tunnel** mode, which requires no domain, no Cloudflare account, and no cost — a fully free alternative for exposing n8n over HTTPS, at the trade-off of a randomly generated, non-persistent URL (see the operational notes at the end of this guide).

## Resolution / Steps

### 1. Install `cloudflared`

```bash
# Debian/Ubuntu
curl -L --output cloudflared.deb https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared.deb

# Verify installation
cloudflared --version
```

### 2. Ensure Docker Is Running

```bash
sudo systemctl start docker
systemctl is-active docker
```

### 3. Start a Quick Cloudflare Tunnel Pointing to n8n's Port

In a **separate terminal**, before starting the n8n container, launch a tunnel targeting the port n8n will listen on:

```bash
cloudflared tunnel --url http://localhost:5678
```

This prints a randomly generated public HTTPS URL, e.g.:

```
https://random-words-here.trycloudflare.com
```

**Note:** This "quick tunnel" mode requires no Cloudflare account or domain — it's ideal for temporary/dev exposure. Keep this terminal open; closing it tears down the tunnel and invalidates the URL.

### 4. Copy the Generated HTTPS URL

Save the `https://...trycloudflare.com` URL from Step 3 — it will be passed into n8n's environment configuration in the next step.

### 5. Start n8n With the Tunnel URL Injected

The critical part: tell n8n its own public address via three environment variables so it generates correct HTTPS webhook URLs instead of `localhost` ones.

```bash
sudo docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -e WEBHOOK_URL="https://random-words-here.trycloudflare.com" \
  -e N8N_WEBHOOK_URL="https://random-words-here.trycloudflare.com" \
  -e N8N_PROTOCOL="https" \
  -v /home/"$USER"/.n8n:/home/node/.n8n \
  n8nio/n8n
```

| Variable | Purpose |
|---|---|
| `WEBHOOK_URL` | Base URL n8n uses to construct webhook links shown in the UI |
| `N8N_WEBHOOK_URL` | Explicit override for the webhook URL n8n registers/exposes |
| `N8N_PROTOCOL` | Tells n8n to treat its external-facing protocol as `https`, not `http` |

### 6. Verify

- Open the tunnel URL (`https://random-words-here.trycloudflare.com`) in a browser — it should load the n8n editor UI over HTTPS.
- Create a new workflow with a Webhook trigger node — the generated webhook URL should show `https://random-words-here.trycloudflare.com/webhook/...`, **not** `http://localhost:5678/...`.
- Trigger the webhook externally (e.g., via `curl` from another machine, or the external service itself) to confirm end-to-end reachability.

### 7. Stopping / Cleanup

Stopping n8n does **not** automatically stop the tunnel — they are independent processes:

```bash
# Stop the n8n container
sudo docker stop n8n

# Stop the tunnel: Ctrl+C in the terminal running `cloudflared tunnel`
```

If the tunnel URL was recorded somewhere (e.g., a local link-tracking file), clear or update that reference once the tunnel is closed, since quick tunnel URLs are **not persistent** — a new one is generated every time `cloudflared tunnel --url ...` is run.

---

## Local-Only Mode (No Tunnel, No HTTPS)

For reference, running without a tunnel — n8n stays on plain HTTP, local-network only, no `WEBHOOK_URL` override needed:

```bash
sudo docker run -d --name n8n \
  -p 5678:5678 \
  -v /home/"$USER"/.n8n:/home/node/.n8n \
  n8nio/n8n
```

Suitable only when no external service needs to reach n8n's webhooks.

---

## Quick Reference

| Command | Purpose |
|---|---|
| `cloudflared tunnel --url http://localhost:5678` | Start a quick HTTPS tunnel to a local port |
| `docker run ... -e N8N_PROTOCOL="https"` | Tell n8n to treat itself as HTTPS-facing |
| `docker run ... -e WEBHOOK_URL="$CF_URL/"` | Set the base URL n8n uses for webhook generation |
| `docker logs -f n8n` | Watch n8n logs for startup/webhook errors |

## Prevention / Operational Notes

- Quick tunnels (`trycloudflare.com`) generate a **new random URL every run** — any external service configured with the old webhook URL will break after a restart. For a stable long-term URL, use a **named Cloudflare Tunnel** bound to a real domain (`cloudflared tunnel create` + DNS route) instead of the quick-tunnel mode.
- Running n8n with `-it --rm` (interactive, auto-remove) is convenient for testing but means the container and its state are destroyed on exit unless the `.n8n` volume is correctly mounted — verify the `-v` volume path persists workflow data across restarts.
- Always confirm `N8N_PROTOCOL=https` is set whenever exposing n8n via any HTTPS-terminating proxy or tunnel — omitting it causes mixed-content and incorrect webhook URL generation even if the tunnel itself is HTTPS.
