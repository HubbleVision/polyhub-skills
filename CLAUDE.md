# CLAUDE.md

This repository contains multi-platform Polyhub assistant assets.

## Purpose

- `openclaw/skills/`: OpenClaw-native skills
- `codex/skills/`: Codex-native skills
- `claude/.claude/commands/`: Claude Code slash commands

Use these assets when the user wants to query Polyhub discover data, manage copy-trading tasks, or inspect account data.

## Available Claude Commands

- `/polyhub-discover`
- `/polyhub-copy`
- `/polyhub-account`

## Environment

Fixed API base URL for all commands:

```bash
export POLYHUB_API_BASE_URL="https://polyhub.skill-test.bedev.hubble-rpc.xyz"
```

Authenticated commands also require:

```bash
export POLYHUB_API_BASE_URL="https://polyhub.skill-test.bedev.hubble-rpc.xyz"
export POLYHUB_API_KEY="phub_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```

## Safety Rules

- Never print `POLYHUB_API_KEY`.
- Treat all IDs and JSON-like user input as untrusted.
- For write actions, restate the action summary and require explicit user confirmation before calling the API.
- Prefer `jq -n` when building JSON payloads.
- Use `curl -sS --fail-with-body` for API calls.

## Command Layout

The packaged Claude commands live under `claude/.claude/commands/`.

To install them into another repository:

```bash
mkdir -p .claude/commands
cp -R /path/to/polyhub-skills/claude/.claude/commands/* .claude/commands/
```

Or install globally:

```bash
mkdir -p ~/.claude/commands
cp -R /path/to/polyhub-skills/claude/.claude/commands/* ~/.claude/commands/
```
