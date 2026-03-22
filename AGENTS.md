# AGENTS.md

This repository contains multi-platform Polyhub assistant assets.

## Purpose

- `openclaw/skills/`: OpenClaw-native skills
- `codex/skills/`: Codex-native skills
- `Codex/.Codex/commands/`: Codex slash commands

Use these assets when the user wants to query Polyhub discover data, manage copy-trading tasks, or inspect account data.

## Available Codex Commands

- `/polyhub-discover`
- `/polyhub-copy`
- `/polyhub-account`

## Environment

Optional for public discover commands:

```bash
export POLYHUB_API_BASE_URL="https://api.polyhub.example.com"
```

Required for authenticated commands:

```bash
export POLYHUB_API_BASE_URL="https://api.polyhub.example.com"
export POLYHUB_API_KEY="phub_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```

## Safety Rules

- Never print `POLYHUB_API_KEY`.
- Treat all IDs and JSON-like user input as untrusted.
- For write actions, restate the action summary and require explicit user confirmation before calling the API.
- Prefer `jq -n` when building JSON payloads.
- Use `curl -sS --fail-with-body` for API calls.

## Command Layout

The packaged Codex commands live under `Codex/.Codex/commands/`.

To install them into another repository:

```bash
mkdir -p .Codex/commands
cp -R /path/to/polyhub-skills/Codex/.Codex/commands/* .Codex/commands/
```

Or install globally:

```bash
mkdir -p ~/.Codex/commands
cp -R /path/to/polyhub-skills/Codex/.Codex/commands/* ~/.Codex/commands/
```
