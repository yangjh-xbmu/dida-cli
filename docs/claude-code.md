# Claude Code And Agent Setup

This page explains how to let Claude Code, Codex, OpenClaw, Hermes, or another
filesystem-capable agent operate Dida365/TickTick through DidaCLI.

Use this page for setup. Use [agent-usage.md](agent-usage.md) for the full
operator policy and command examples.

## Recommended Setup

Install DidaCLI first and make sure the `dida` binary is on `PATH`:

```bash
dida version
dida doctor --json
```

Then install the repo-local skill:

```text
skills/dida-cli/SKILL.md
```

For Claude Code on Windows PowerShell:

```powershell
New-Item -ItemType Directory -Force $env:USERPROFILE\.claude\skills | Out-Null
Copy-Item -Recurse .\skills\dida-cli $env:USERPROFILE\.claude\skills\dida-cli -Force
```

For Codex on Windows PowerShell:

```powershell
New-Item -ItemType Directory -Force $env:USERPROFILE\.codex\skills | Out-Null
Copy-Item -Recurse .\skills\dida-cli $env:USERPROFILE\.codex\skills\dida-cli -Force
```

See [skill-installation.md](skill-installation.md) for symlink and other agent
options.

## Authentication

Authentication stays local. Agents must not ask users to paste cookies or
tokens into chat.

For the Web API channel, ask the operator to run:

```bash
dida auth login --browser --json
dida auth status --verify --json
```

For the official MCP channel, ask the operator to run:

```bash
dida official token set --token-stdin --json
dida official token status --json
```

For official OpenAPI, ask the operator to configure the OAuth client and run:

```bash
dida openapi client set --id <client-id> --secret-stdin --json
dida openapi login --browser --json
dida openapi status --json
```

These channels are independent. A Web API browser cookie is not an OpenAPI
bearer token, and an MCP token is not an OpenAPI token.

## Agent Bootstrap Prompt

Paste this into a Claude Code, Codex, or similar agent session when the skill is
not installed yet:

```text
Use DidaCLI for Dida365/TickTick work.

Start with:
  dida doctor --json
  dida schema list --compact --json
  dida channel list --json
  dida agent context --outline --json

Do not ask me to paste cookies or tokens into chat. If auth is missing, tell me
the exact local command to run.

Use --json for all commands. Prefer --compact or --outline for broad reads.
Before any write, run the same command with --dry-run and show the preview.
Do not execute delete or merge operations unless I explicitly approve the
specific target and the final command includes --yes.

Use exact IDs from command output. Do not guess project, task, folder, tag,
comment, habit, or focus IDs from names.

Keep Web API, Official MCP, and Official OpenAPI commands separate.
```

## First Agent Commands

After setup, an agent should run these local and read-only checks:

```bash
dida doctor --json
dida schema list --compact --json
dida channel list --json
dida auth status --verify --json
dida agent context --outline --json
```

If the goal is only to read tasks, prefer bounded compact output:

```bash
dida task today --compact --json
dida task upcoming --days 14 --limit 50 --compact --json
dida task search --query <text> --limit 20 --compact --json
dida project list --json
```

Use full task data only when descriptions, checklist items, reminders, or raw
fields are actually needed:

```bash
dida task get <task-id> --json
```

## Write Policy

Agents should preview generated writes first:

```bash
dida task create --project <project-id> --title "Example" --dry-run --json
dida task update <task-id> --project <project-id> --title "New title" --dry-run --json
dida comment create --project <project-id> --task <task-id> --text "Example" --dry-run --json
```

Then execute only after the operator accepts the preview:

```bash
dida task create --project <project-id> --title "Example" --json
```

Destructive commands require explicit operator approval and `--yes`:

```bash
dida task delete <task-id> --project <project-id> --dry-run --json
dida task delete <task-id> --project <project-id> --yes --json
```

Do not use `dida official call` for write-capable MCP tools unless the operator
explicitly approves the exact tool name and payload. `official call` has no
DidaCLI dry-run layer.

## What Agents Should Read In This Repo

For task operation policy:

```text
docs/agent-usage.md
```

For short JSON-first command examples:

```text
docs/llm-quickstart.md
```

For install instructions:

```text
docs/skill-installation.md
```

For machine-readable command contracts:

```bash
dida schema list --compact --json
dida schema show <schema-id> --json
```

For the installable skill body:

```text
skills/dida-cli/SKILL.md
```

## Safety Boundaries

- Do not paste or print cookie values, OAuth secrets, MCP tokens, or raw `t=`
  cookie values.
- Do not commit files from `~/.dida-cli/`.
- Do not use raw probes for writes. `dida raw get` is read-only by design.
- Do not guess IDs from visible names when command output can provide exact IDs.
- Do not create disposable habits, focus records, projects, or tasks only to
  test write paths unless the operator has approved the cleanup plan.
