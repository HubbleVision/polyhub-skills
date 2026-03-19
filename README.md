# polyhub-skills

OpenClaw Skills for [Polyhub](https://github.com/HubbleVision/polyhub) — public discover queries plus authenticated copy-trading and account operations.

## Skills

| Skill | Description |
|-------|-------------|
| `polyhub_discover` | Public discover page queries: tags, trader rankings, cross-tag filters, trader-by-address lookup, market tag lookup |
| `polyhub_copy` | Copy-trading task management: CRUD, positions, trades, sell, batch ops, signals, stream, TPSL rules, safer JSON payload templates |
| `polyhub_account` | Account overview: portfolio stats, fee history, manual order placement with explicit field validation and order-type guidance |

## Quick Start

### Public discover skill

`polyhub_discover` only needs:

```bash
export POLYHUB_API_BASE_URL="https://api.polyhub.example.com"
```

### Authenticated skills

`polyhub_copy` and `polyhub_account` also require an API key:

1. Get a Polyhub API key (prefix `phub_`)
   - Recommended: open PolyHub Web, click the avatar menu, then open `Skills API Key`
   - Click `申请 API Key` and copy the plaintext key immediately. It is shown only once
2. Set environment variables:
   ```bash
   export POLYHUB_API_BASE_URL="https://api.polyhub.example.com"
   export POLYHUB_API_KEY="phub_..."
   ```
3. Install skills — see [OPENCLAW.md](OPENCLAW.md) for detailed instructions

## Web Flow

PolyHub Web now provides a dedicated `Skills API Key` entry for OpenClaw skills:

1. Open the avatar dropdown in `poly_copy`
2. Click `Skills API Key`
3. Click `申请 API Key`
4. Copy the generated `phub_` key and set:
   ```bash
   export POLYHUB_API_BASE_URL="https://your-polyhub-host/api/v1"
   export POLYHUB_API_KEY="phub_..."
   ```
5. Continue with the installation steps in [OPENCLAW.md](OPENCLAW.md)

## Directory Structure

```
openclaw/
  skills/
    polyhub_discover/SKILL.md  # Public discover queries
    polyhub_copy/SKILL.md      # Copy-trading management
    polyhub_account/SKILL.md   # Account & trading
OPENCLAW.md                    # Installation & usage guide
```
