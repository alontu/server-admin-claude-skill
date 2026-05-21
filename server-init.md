Run a structured server discovery wizard, then generate two files tailored to the target server:
1. `CLAUDE.md` — operational context for Claude Code sessions
2. `{{HOSTNAME}}_user_manual.md` — human-facing operational reference

The output captures server-specific facts so any future Claude Code session in that directory immediately has full operational context, and the operator has a ready reference for day-to-day tasks.

## Wizard Protocol

- Conduct exactly 4 rounds, maximum 2 questions per round
- Number each round: "Round N of 4"
- Give a one-sentence acknowledgment of the user's answers before moving to the next round
- After Round 4, present a **Server Summary** and ask for confirmation or corrections
- Generate both output files in the current working directory only after the user confirms the summary

---

## Round 1 — Identity

Ask:
1. What is the server's hostname and hardware specs (CPU, RAM, storage)?
2. What OS and version is it running, and what is its primary role or purpose?

## Round 2 — Network & Access

Ask:
1. What are the network interfaces and their IPs? Is Tailscale, WireGuard, or any SSH tunnel in use?
2. Any network topology quirks to note? (e.g., broken IPv6, mesh Wi-Fi not bridging subnets, hairpin NAT, etc.)

## Round 3 — Users & Services

Ask:
1. What user accounts exist? For each: role, and sudo level (NOPASSWD / password required / none).
2. What services are running? List Docker containers and native systemd services with their ports. Any web UIs?

## Round 4 — Ops Patterns

Ask:
1. Where are configs, data volumes, and backups stored? Note any non-standard paths.
2. Any known failure modes, gotchas, or workarounds specific to this server?

---

## Server Summary

After Round 4, present a numbered list of all collected facts and ask:

> "Does this look right? Correct or add anything before I generate the files."

Wait for confirmation before proceeding.

---

## Generate Files

After the user confirms, write both files to the current working directory.

**Substitution rules (apply to both templates):**
- Replace every `{{PLACEHOLDER}}` with the collected value
- If a section has no data, write `TBD` or omit the section — never invent values
- For multi-row tables, expand the row pattern for each collected item
- Include `[if applicable]` sections only when relevant data was collected
- Keep methodology sections (Common Commands, User Manual Maintenance) verbatim

---

## File 1: CLAUDE.md Template

```markdown
# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Project Overview

This is an infrastructure documentation project for **{{HOSTNAME}}** — {{SERVER_ROLE}}.

**Hardware:** {{HARDWARE}}
**OS:** {{OS_VERSION}}

## User Accounts

| User | Purpose | Sudo |
|---|---|---|
| `{{USER_1_NAME}}` | {{USER_1_ROLE}} | {{USER_1_SUDO}} |

Use `{{ADMIN_USER}}` for all admin/infra work.

## Remote Access

```bash
# Local LAN
ssh {{ADMIN_USER}}@{{HOSTNAME}}.local
# Static IP: ssh {{ADMIN_USER}}@{{LAN_IP}}
```

[if applicable] Tailscale:
```bash
ssh {{ADMIN_USER}}@{{TAILSCALE_HOSTNAME}}   # MagicDNS
ssh {{ADMIN_USER}}@{{TAILSCALE_IP}}
```

[if applicable] Tunnel ({{TUNNEL_TYPE}}):
```bash
{{TUNNEL_SSH_COMMAND}}
```

Web UIs:
- **{{SERVICE_NAME}}**: {{SERVICE_URL}}

## Server Architecture

**Network Stack:**
- Interface: {{NETWORK_INTERFACE}} — {{NETWORK_IP}}
- Manager: {{NETWORK_MANAGER}}
[if applicable] Network quirks: {{NETWORK_QUIRKS}}

**Container Runtime:**
{{CONTAINER_RUNTIME}}

**Storage Layout:**
- Configs: `{{CONFIG_PATH}}`
- Data: `{{DATA_PATH}}`
- Backups: `{{BACKUP_PATH}}`

## Services

| Service | Type | Port / URL | Notes |
|---|---|---|---|
| {{SERVICE_NAME}} | {{docker/systemd}} | {{PORT}} | {{NOTES}} |

## Common Commands

```bash
# Network status
ip -br addr show
ip route

# Docker
docker ps
docker compose -f {{COMPOSE_BASE_PATH}}/<service>/docker-compose.yml {logs,restart,up,down}

# Systemd
sudo systemctl {start,stop,restart,status} <service>
journalctl -u <service> -n 100 --no-pager
```

## Known Issues & Workarounds

{{KNOWN_ISSUES}}

## Current State

**Completed:**
- OS, SSH, networking baseline
{{ADDITIONAL_COMPLETED_ITEMS}}

**Pending:**
- Backup strategy
- Review and extend this CLAUDE.md after initial setup is stable

## User Manual Maintenance

**IMPORTANT:** Maintain `{{HOSTNAME}}_user_manual.md` as the authoritative reference for the server operator.

When making changes to the server (installing services, modifying configuration, adding containers, changing network settings):
1. Update the user manual to reflect the current state
2. Include: service URLs, ports, credentials location, start/stop commands, backup procedures
3. Document any manual steps the operator needs to perform
4. Keep the manual focused on operational tasks, not implementation history

The manual should answer: "How do I access X?", "How do I restart Y?", "Where are the configs for Z?"
```

---

## File 2: {{HOSTNAME}}_user_manual.md Template

```markdown
# {{HOSTNAME}} — Operator Manual

> Authoritative reference for day-to-day server management.  
> Last updated: {{DATE}}

---

## Quick Reference

| What | Where |
|---|---|
| LAN SSH | `ssh {{ADMIN_USER}}@{{HOSTNAME}}.local` or `{{ADMIN_USER}}@{{LAN_IP}}` |
[if applicable] | Tailscale SSH | `ssh {{ADMIN_USER}}@{{TAILSCALE_HOSTNAME}}` |
[if applicable] | Tunnel SSH | `{{TUNNEL_SSH_COMMAND}}` |
| Config files | `{{CONFIG_PATH}}` |
| Data volumes | `{{DATA_PATH}}` |
| Backups | `{{BACKUP_PATH}}` |

### Service URLs

| Service | URL | Notes |
|---|---|---|
| {{SERVICE_NAME}} | {{SERVICE_URL}} | {{NOTES}} |

---

## Accessing the Server

### SSH — Local Network
```bash
ssh {{ADMIN_USER}}@{{HOSTNAME}}.local
# or by IP
ssh {{ADMIN_USER}}@{{LAN_IP}}
```

[if applicable]
### SSH — Tailscale (any network)
```bash
ssh {{ADMIN_USER}}@{{TAILSCALE_HOSTNAME}}
ssh {{ADMIN_USER}}@{{TAILSCALE_IP}}
```

[if applicable]
### SSH — {{TUNNEL_TYPE}} Tunnel (away from home, no Tailscale)
```bash
{{TUNNEL_SSH_COMMAND}}
```

---

## Managing Services

### Docker Services

```bash
# List running containers
docker ps

# View logs
docker compose -f {{COMPOSE_BASE_PATH}}/<service>/docker-compose.yml logs -f

# Restart a service
docker compose -f {{COMPOSE_BASE_PATH}}/<service>/docker-compose.yml restart

# Stop / start
docker compose -f {{COMPOSE_BASE_PATH}}/<service>/docker-compose.yml down
docker compose -f {{COMPOSE_BASE_PATH}}/<service>/docker-compose.yml up -d
```

### Systemd Services

```bash
# Status
sudo systemctl status <service>

# Restart
sudo systemctl restart <service>

# Logs (last 100 lines)
journalctl -u <service> -n 100 --no-pager

# Follow live logs
journalctl -u <service> -f
```

### Service-Specific Commands

[Expand one subsection per service collected in the wizard. Each subsection should include: start/stop/restart command, log command, config file path, and any service-specific gotchas from Round 4.]

#### {{SERVICE_NAME}}

```bash
# Status
{{SERVICE_STATUS_COMMAND}}

# Restart
{{SERVICE_RESTART_COMMAND}}

# Logs
{{SERVICE_LOG_COMMAND}}
```

Config: `{{SERVICE_CONFIG_PATH}}`
{{#if SERVICE_GOTCHA}}> Note: {{SERVICE_GOTCHA}}{{/if}}

---

## Network

```bash
# Show interfaces and IPs
ip -br addr show

# Show routing table
ip route
```

[if applicable]
### Applying Network Config Changes
```bash
sudo netplan try && sudo netplan apply
```

---

## Storage & Backups

| Path | Purpose |
|---|---|
| `{{CONFIG_PATH}}` | Service configuration files |
| `{{DATA_PATH}}` | Persistent service data |
| `{{BACKUP_PATH}}` | Backups |

### Creating a Manual Backup
```bash
# Example: backup a service config directory
sudo tar -czf {{BACKUP_PATH}}/{{SERVICE_NAME}}-$(date +%Y%m%d).tar.gz {{CONFIG_PATH}}/{{SERVICE_NAME}}
```

---

## Troubleshooting

### Server Unreachable
1. Check if host responds: `ping {{HOSTNAME}}.local`
2. Try static IP: `ping {{LAN_IP}}`
[if applicable] 3. Try Tailscale: `ping {{TAILSCALE_IP}}`
4. Check network interfaces: `ip -br addr show` (run on server console if accessible)

### Service Not Responding
1. Check container/unit is running: `docker ps` or `systemctl status <service>`
2. Check logs for errors: `journalctl -u <service> -n 50 --no-pager`
3. Restart: `sudo systemctl restart <service>` or `docker compose ... restart`

### Known Issues

{{KNOWN_ISSUES}}

---

## User Accounts

| User | Purpose | Sudo |
|---|---|---|
| `{{USER_1_NAME}}` | {{USER_1_ROLE}} | {{USER_1_SUDO}} |

> Use `{{ADMIN_USER}}` for all manual admin tasks.

---

*Keep this manual updated whenever the server configuration changes.*
```
