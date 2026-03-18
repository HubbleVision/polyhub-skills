# polyhub-skills

OpenClaw Skills for [Polyhub](https://github.com/HubbleVision/polyhub) — copy-trading management and account operations via API Key.

## Skills

| Skill | Description |
|-------|-------------|
| `polyhub_copy` | Copy-trading task management: CRUD, positions, trades, sell, batch ops, signals, stream, TPSL rules, safer JSON payload templates |
| `polyhub_account` | Account overview: portfolio stats, fee history, manual order placement with explicit field validation and order-type guidance |

## Quick Start

1. Get a Polyhub API key (prefix `phub_`)
2. Set environment variables:
   ```bash
   export POLYHUB_API_BASE_URL="https://api.polyhub.example.com"
   export POLYHUB_API_KEY="phub_..."
   ```
3. Install skills — see [OPENCLAW.md](OPENCLAW.md) for detailed instructions

## Directory Structure

```
openclaw/
  skills/
    polyhub_copy/SKILL.md      # Copy-trading management
    polyhub_account/SKILL.md   # Account & trading
OPENCLAW.md                    # Installation & usage guide
```
