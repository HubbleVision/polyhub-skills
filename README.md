# PolyHub Skills 🦞

Control your Polymarket copy-trading account from any AI you already use.

We spent months tracking who actually wins on Polymarket -
who's trading, which markets, what edge.
These Skills put that data and execution layer directly inside your AI agent.

## What you can do

Ask your AI to find the top-performing traders on Polymarket.

Ask it to copy one. Ask it how your portfolio is doing.

It handles the rest.

- 🔍 **polyhub_discover** - Browse trader leaderboards, filter by tag, look up any address
- 📋 **polyhub_copy** - Create, pause, delete copy trades. Track positions, signals, trades in real time
- 📊 **polyhub_account** - Portfolio stats, fee history, manual order placement

Works with **OpenClaw · Claude Code · Codex** - and any platform that supports Skills.

> **Telegram & WeChat users:** PolyHub Skills are available on Telegram and WeChat
> via OpenClaw. Once you have OpenClaw set up, you can chat with your agent
> and control your PolyHub account directly from either app - no extra configuration needed.

---

## Quick Start

### Discover (no setup needed)

`polyhub_discover` works out of the box - no API key required.

```bash
# Default host
export POLYHUB_API_BASE_URL="https://polyhub.skill-test.bedev.hubble-rpc.xyz"
```

### Copy Trading & Account (requires API key)

1. Open PolyHub Web -> avatar menu -> **Skills API Key**
2. Click **Request API Key** and copy it immediately (shown once)
3. Set environment variables:

```bash
export POLYHUB_API_BASE_URL="https://polyhub.skill-test.bedev.hubble-rpc.xyz"
export POLYHUB_API_KEY="phub_..."
```

4. Install for your platform:
   - OpenClaw -> see [**OPENCLAW.md**](OPENCLAW.md)
   - Codex -> see [**CODEX.md**](CODEX.md)
   - Claude Code -> see [**CLAUDE.md**](CLAUDE.md)

---

## Platform Guides

| Platform | Guide |
| --- | --- |
| OpenClaw | [OPENCLAW.md](OPENCLAW.md) |
| Codex | [CODEX.md](CODEX.md) |
| Claude Code | [CLAUDE.md](CLAUDE.md) |

---

## Directory Structure

```text
openclaw/skills/
  polyhub_discover/SKILL.md
  polyhub_copy/SKILL.md
  polyhub_account/SKILL.md
codex/skills/
  polyhub-discover/SKILL.md
  polyhub-copy/SKILL.md
  polyhub-account/SKILL.md
claude/.claude/commands/
  polyhub-discover.md
  polyhub-copy.md
  polyhub-account.md
```

---

Built by [PolyHub](https://x.com/MeetPolyHub) × [Hubble](https://hubble.xyz/) 🦞
