# DNS Resolution Issues

## Problem / Context

A host or application fails to resolve a domain name to an IP address, resulting in errors such as `Could not resolve host`, `NXDOMAIN`, `ERR_NAME_NOT_RESOLVED`, or connection timeouts despite the target service being reachable and online. This affects browsing, package managers (`apt`, `npm`, `pip`), Git operations, and container networking alike.

Typical symptoms:
- `ping google.com` fails with `unknown host`, but `ping 8.8.8.8` succeeds (confirms DNS-specific failure, not a general connectivity loss).
- Intermittent resolution — works sometimes, fails other times.
- Resolution works on the host but fails inside a container or VM.
- Corporate/VPN network resolves internal hostnames but not public domains, or vice versa.

## Root Cause

DNS resolution failures generally stem from one of the following:

1. **Misconfigured or unreachable DNS resolver** — `/etc/resolv.conf` (Linux) or network adapter DNS settings point to a DNS server that is down, blocked, or incorrect.
2. **Stale local DNS cache** — the OS or browser cached a stale/negative response (e.g., a previous `NXDOMAIN`) and continues serving it despite the record now existing.
3. **`/etc/hosts` conflicts** — a manual entry overrides the domain with an incorrect or outdated IP.
4. **Split-horizon DNS / VPN interference** — a VPN or corporate network redirects DNS queries to an internal resolver that cannot resolve public domains (or vice versa).
5. **Container/VM network isolation** — the container's network namespace does not inherit the host's DNS configuration, or Docker's embedded DNS (`127.0.0.11`) cannot reach an upstream resolver.
6. **Firewall/security software blocking DNS traffic** — outbound UDP/TCP port 53 (or 853 for DNS-over-TLS) is blocked.
7. **Domain-side issues** — the domain's authoritative nameservers are misconfigured, expired, or the record itself doesn't exist (external to the local environment).

## Resolution / Steps

### 1. Confirm the Failure Is DNS-Specific

Isolate whether the issue is DNS or general connectivity:

```bash
# Bypasses DNS entirely — tests raw IP connectivity
ping 8.8.8.8

# Uses DNS — tests name resolution
ping google.com
```

If the IP succeeds but the domain fails, the problem is confirmed to be DNS resolution.

### 2. Test Resolution Directly Against a Known-Good Resolver

Bypass the system resolver to check if the issue is with the configured DNS server itself:

```bash
# Query Google's public DNS directly
nslookup google.com 8.8.8.8

# Or using dig (more detailed output)
dig google.com @8.8.8.8
```

If this succeeds but the default system query fails, the configured local/ISP resolver is the problem.

### 3. Inspect Current DNS Configuration

**Linux:**
```bash
cat /etc/resolv.conf
```

**Check via systemd-resolved (modern Ubuntu/Debian):**
```bash
resolvectl status
```

**Windows:**
```powershell
ipconfig /all
```

Verify the `nameserver` entries point to a valid, reachable DNS server (e.g., `8.8.8.8`, `1.1.1.1`, or your internal corporate resolver).

### 4. Flush the Local DNS Cache

Stale cached records are a common cause of "it works for others but not me."

**Linux (systemd-resolved):**
```bash
sudo resolvectl flush-caches
```

**macOS:**
```bash
sudo dscacheutil -flushcache
sudo killall -HUP mDNSResponder
```

**Windows:**
```powershell
ipconfig /flushdns
```

### 5. Check `/etc/hosts` for Conflicting Entries

```bash
cat /etc/hosts
```

Remove or correct any stale manual entry that overrides the domain in question.

### 6. Set a Reliable Fallback DNS Server

If the default resolver (ISP or DHCP-assigned) is unreliable, override it with a public resolver:

```bash
# Temporary override — edit /etc/resolv.conf directly (may be overwritten by DHCP/NetworkManager)
sudo tee /etc/resolv.conf <<EOF
nameserver 1.1.1.1
nameserver 8.8.8.8
EOF
```

For a persistent fix on systemd-resolved-based systems, edit `/etc/systemd/resolved.conf`:

```ini
[Resolve]
DNS=1.1.1.1 8.8.8.8
```

Then restart the service:

```bash
sudo systemctl restart systemd-resolved
```

### 7. Fix DNS Resolution Inside Docker Containers

If resolution fails only inside a container:

```bash
# Test resolution from within the running container
docker exec -it <container_name> nslookup google.com
```

Explicitly set a DNS server for the container:

```yaml
# docker-compose.yml
services:
  app:
    dns:
      - 8.8.8.8
      - 1.1.1.1
```

Or globally in Docker's daemon config (`/etc/docker/daemon.json`):

```json
{
  "dns": ["8.8.8.8", "1.1.1.1"]
}
```

Restart Docker after changes:

```bash
sudo systemctl restart docker
```

### 8. Check for VPN/Split-Tunnel Interference

If the issue only occurs while connected to a VPN, temporarily disconnect and retest. If resolution succeeds without the VPN, the VPN's assigned DNS server is likely misrouting queries — check the VPN client's DNS settings or split-tunneling configuration.

### 9. Verify the Domain Itself (External Cause)

Rule out a problem with the domain's own DNS records:

```bash
dig google.com +trace
```

Or check via a third-party tool (e.g., an online DNS propagation checker) if the domain was recently configured or migrated.

---

## Quick Diagnostic Commands

| Command | Purpose |
|---|---|
| `ping <ip>` | Test raw connectivity, bypassing DNS |
| `nslookup <domain> <dns-server>` | Query a specific DNS server directly |
| `dig <domain>` | Detailed DNS query with full record data |
| `dig <domain> +trace` | Trace resolution path from root nameservers |
| `resolvectl status` | View active DNS configuration (systemd-resolved) |
| `cat /etc/resolv.conf` | View configured nameservers (Linux) |
| `ipconfig /flushdns` | Flush DNS cache (Windows) |
| `sudo resolvectl flush-caches` | Flush DNS cache (Linux/systemd-resolved) |

## Prevention Tips

- Prefer reliable public resolvers (`1.1.1.1`, `8.8.8.8`) or a properly maintained internal DNS server over unmanaged ISP defaults.
- Avoid hardcoding entries in `/etc/hosts` unless necessary for local development — they're a common source of "forgotten" stale overrides.
- Document any VPN-specific DNS behavior for the team to avoid repeated confusion.
- For containerized environments, standardize DNS settings in `docker-compose.yml` or the Docker daemon config rather than relying on host inheritance.
