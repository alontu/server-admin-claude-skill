# claude-server-init

A Claude Code custom command that runs an interactive wizard to bootstrap server management documentation. Point it at a new server's project directory and it interviews you about the server, then generates two ready-to-use files.

## What it generates

| File | Purpose |
|---|---|
| `CLAUDE.md` | Operational context injected into every Claude Code session in this directory — services, IPs, user accounts, known quirks |
| `{{hostname}}_user_manual.md` | Human-facing operator reference — how to access, restart, debug, and back up everything on the server |

## Installation

Copy `server-init.md` to your Claude Code global commands directory:

```bash
# macOS / Linux
mkdir -p ~/.claude/commands
cp server-init.md ~/.claude/commands/server-init.md

# Windows (PowerShell)
New-Item -ItemType Directory -Force "$env:USERPROFILE\.claude\commands"
Copy-Item server-init.md "$env:USERPROFILE\.claude\commands\server-init.md"
```

The command is immediately available in any Claude Code session — no restart needed.

## Usage

Navigate to the directory where you manage the server (or create one), then run:

```
/server-init
```

Claude will run a 4-round discovery wizard (~10 minutes), show you a summary for confirmation, then write both files to the current directory.

## Wizard rounds

| Round | Topics |
|---|---|
| 1 | Server hostname, hardware, OS, role |
| 2 | Network interfaces, IPs, Tailscale/tunnel, topology quirks |
| 3 | User accounts + sudo levels, services, ports, web UIs |
| 4 | Config/data/backup paths, known failure modes and workarounds |

## Example output

After running `/server-init` on a home server called `my-server`:

```
my-server-docs/
  CLAUDE.md                    ← Claude picks this up automatically on next session
  my-server_user_manual.md     ← Operator reference: access, service commands, troubleshooting
```

## Design principles

- **Facts, not methodology.** The wizard collects server-specific data. Ops discipline (confirm before destructive actions, diagnose root causes, etc.) lives in your global `~/.claude/CLAUDE.md` and applies everywhere.
- **No invented values.** If a piece of info wasn't provided in the wizard, the template writes `TBD` rather than guessing.
- **Two audiences.** `CLAUDE.md` is written for Claude. The user manual is written for the human operator. Same facts, different framing.
- **Living documents.** Both files are meant to be updated as the server evolves. The `CLAUDE.md` includes a maintenance reminder for itself.

## Updating an existing server's docs

Re-run `/server-init` — Claude will overwrite both files with fresh content. Or edit the generated files directly; they're plain markdown.

## Requirements

- [Claude Code](https://claude.ai/code) CLI, desktop app, or IDE extension
- Any Claude model (Haiku and above work well for the wizard)
