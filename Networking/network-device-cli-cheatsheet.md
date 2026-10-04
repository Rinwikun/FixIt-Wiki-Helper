# Network Device CLI Cheat Sheet (Cisco IOS, Junos, MikroTik, Arista)

## Problem / Context

Network equipment runs its own vendor-specific operating system with a dedicated command-line interface — these are **not** general-purpose Unix/Windows shells, but closed, purpose-built CLIs for configuring routers, switches, and firewalls. Each vendor uses different syntax, privilege models, and configuration paradigms, which is a common source of confusion when moving between platforms. This cheat sheet covers the most commonly used commands across four major platforms.

## Root Cause

Not applicable — this is a reference guide rather than a troubleshooting document.

## Resolution / Steps

### Cisco IOS CLI

**Privilege levels & modes:**
```
Router>                    # User EXEC mode (limited, read-only)
Router> enable
Router#                    # Privileged EXEC mode
Router# configure terminal
Router(config)#            # Global configuration mode
Router(config)# interface GigabitEthernet0/1
Router(config-if)#         # Interface configuration mode
```

**Common commands:**

| Command | Description |
|---|---|
| `show running-config` | Display the currently active configuration |
| `show interfaces` | Display interface status and statistics |
| `show ip route` | Display the routing table |
| `show version` | Display IOS version and hardware info |
| `copy running-config startup-config` | Save the running config so it persists across reboot |
| `interface GigabitEthernet0/1` → `ip address <ip> <mask>` | Assign an IP address to an interface |
| `no shutdown` | Enable an interface (interfaces are admin-down by default on many platforms) |
| `ping <ip>` / `traceroute <ip>` | Standard connectivity diagnostics |

### Junos CLI (Juniper)

**Modes:**
```
user@router>                      # Operational mode
user@router> configure
user@router#                      # Configuration mode
```

**Common commands:**

| Command | Description |
|---|---|
| `show configuration` | Display the current configuration (operational mode) |
| `show interfaces terminal` | Display interface status |
| `show route` | Display the routing table |
| `show version` | Display Junos version and hardware info |
| `set interfaces ge-0/0/0 unit 0 family inet address <ip>/<prefix>` | Assign an IP address to an interface (configuration mode) |
| `commit` | Apply and activate staged configuration changes |
| `commit confirmed` | Apply changes with automatic rollback if not confirmed within a time window — a safety net for remote changes |
| `rollback <n>` | Revert to a previous committed configuration version |

**Key Junos concept:** unlike Cisco IOS (which applies config line-by-line immediately), Junos uses a **candidate configuration** model — changes are staged and only take effect after an explicit `commit`.

### MikroTik RouterOS CLI

**Navigation:**
```
[admin@MikroTik] > /interface print
[admin@MikroTik] > /ip address print
```

**Common commands:**

| Command | Description |
|---|---|
| `/interface print` | List all interfaces and their status |
| `/ip address print` | List configured IP addresses |
| `/ip address add address=<ip>/<prefix> interface=<name>` | Assign an IP address to an interface |
| `/ip route print` | Display the routing table |
| `/system resource print` | Display CPU, memory, and uptime info |
| `/export` | Export the full configuration as a script (useful for backup/review) |
| `/system backup save` | Save a full binary system backup |
| `/ping <ip>` | Connectivity test |

**Key RouterOS concept:** commands are organized hierarchically by path (`/interface`, `/ip`, `/system`), navigable either by full path per command or by entering a submenu context.

### Arista EOS CLI

Arista EOS deliberately mirrors Cisco IOS syntax closely, easing migration between the two:

```
switch>                    # User EXEC mode
switch> enable
switch#                    # Privileged EXEC mode
switch# configure terminal
switch(config)#            # Global configuration mode
```

**Common commands:**

| Command | Description |
|---|---|
| `show running-config` | Display the currently active configuration |
| `show interfaces status` | Display interface status summary |
| `show ip route` | Display the routing table |
| `show version` | Display EOS version and hardware info |
| `copy running-config startup-config` | Save the running config so it persists across reboot |
| `bash` | Drop into a genuine Linux Bash shell underneath EOS (Arista EOS runs on a real Linux kernel — a notable differentiator from the other three platforms) |

---

## Quick Comparison Table

| Aspect | Cisco IOS | Junos | MikroTik RouterOS | Arista EOS |
|---|---|---|---|---|
| Config application | Immediate, line-by-line | Staged, requires `commit` | Immediate | Immediate, line-by-line |
| Save-to-persist command | `copy running-config startup-config` | Implicit via `commit` | `/system backup save` or auto-persisted | `copy running-config startup-config` |
| Underlying OS | Proprietary (IOS) / Linux (IOS XE) | FreeBSD-based | Linux-based | Linux-based (exposed via `bash`) |
| Rollback support | Limited (manual) | Native (`rollback <n>`) | Manual (restore backup) | Limited (manual) |

## Prevention Tips

- On Junos, never forget `commit` — changes typed in configuration mode have **no effect** on the live device until committed, which surprises engineers coming from Cisco IOS's immediate-apply model.
- Use `commit confirmed` on Junos (or equivalent safety mechanisms where available) when making remote changes over a connection that the change itself might break — it auto-reverts if you lose access and don't confirm in time.
- Always save/persist configuration explicitly after changes on platforms that don't auto-persist (Cisco IOS, Arista EOS) — a power cycle without saving reverts to the last saved startup configuration, discarding all unsaved changes.
