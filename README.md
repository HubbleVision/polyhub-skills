# polyhub-skills

Multi-platform Polyhub assistant assets for [Polyhub](https://github.com/HubbleVision/polyhub) — public discover queries plus authenticated copy-trading and account operations.

## Skills

| Capability | OpenClaw | Codex | Claude | Description |
|------------|----------|-------|--------|-------------|
| Discover | `polyhub_discover` | `polyhub-discover` | `/polyhub-discover` | Public discover page queries: tags, trader rankings, cross-tag filters, trader-by-address lookup, market tag lookup |
| Copy Trading | `polyhub_copy` | `polyhub-copy` | `/polyhub-copy` | Copy-trading task management: CRUD, positions, trades, sell, batch ops, signals, stream, TPSL rules, safer JSON payload templates |
| Account | `polyhub_account` | `polyhub-account` | `/polyhub-account` | Account overview: portfolio stats, fee history, manual order placement with explicit field validation and order-type guidance |

`portfolio/stats` semantics are aligned with current Polyhub UI usage: official `positionsValue`, official `totalPnL`, and `availableBalance` already net of `unsettledFees`.

## Quick Start

### Public discover skill

`polyhub_discover` can be used directly without any environment variables.

Default API host:

```bash
https://api.polyhub.example.com
```

Optional override for custom deployments:

```bash
export POLYHUB_API_BASE_URL="https://api.polyhub.example.com"
```

### Authenticated skills

`polyhub_copy` and `polyhub_account` require both base URL and API key:

1. Get a Polyhub API key (prefix `phub_`)
   - Recommended: open PolyHub Web, click the avatar menu, then open `Skills API Key`
   - Click `申请 API Key` and copy the plaintext key immediately. It is shown only once
2. Set environment variables:
   ```bash
   export POLYHUB_API_BASE_URL="https://api.polyhub.example.com"
   export POLYHUB_API_KEY="phub_..."
   ```
3. Install skills
   - OpenClaw: see [OPENCLAW.md](OPENCLAW.md)
   - Codex: see [CODEX.md](CODEX.md)
   - Claude Code: copy the packaged commands from `claude/.claude/commands/` or read [CLAUDE.md](CLAUDE.md)

## Web Flow

PolyHub Web now provides a dedicated `Skills API Key` entry that can be reused by OpenClaw, Codex, and Claude Code assets:

1. Open the avatar dropdown in `poly_copy`
2. Click `Skills API Key`
3. Click `申请 API Key`
4. Copy the generated `phub_` key and set:
   ```bash
   export POLYHUB_API_BASE_URL="https://your-polyhub-host/api/v1"
   export POLYHUB_API_KEY="phub_..."
   ```
5. Continue with the platform-specific installation steps in [OPENCLAW.md](OPENCLAW.md), [CODEX.md](CODEX.md), or [CLAUDE.md](CLAUDE.md)

## Directory Structure

```
openclaw/
  skills/
    polyhub_discover/SKILL.md  # Public discover queries
    polyhub_copy/SKILL.md      # Copy-trading management
    polyhub_account/SKILL.md   # Account & trading
codex/
  skills/
    polyhub-discover/SKILL.md  # Codex discover skill
    polyhub-copy/SKILL.md      # Codex copy-trading skill
    polyhub-account/SKILL.md   # Codex account skill
claude/
  .claude/
    commands/
      polyhub-discover.md      # Claude discover command
      polyhub-copy.md          # Claude copy-trading command
      polyhub-account.md       # Claude account command
OPENCLAW.md                    # OpenClaw installation & usage guide
CODEX.md                       # Codex installation & usage guide
CLAUDE.md                      # Claude Code usage guide
```
