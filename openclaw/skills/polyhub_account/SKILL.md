---
name: polyhub_account
description: View portfolio stats on Polyhub using an API key.
---

# Polyhub Account Skill

Version: v0.3.5

## When to use

Use this skill when the user asks about:

- Portfolio overview or stats

## Requirements

- `POLYHUB_API_BASE_URL` — Polyhub API server URL (e.g. `https://api.polyhub.example.com`)
- `POLYHUB_API_KEY` — API key (must start with `phub_`)
- `curl` must be available in the runtime environment.
- `jq` is strongly recommended for building JSON payloads safely.

## Safety rules

- Never print `POLYHUB_API_KEY` in the output.
- Prefer building JSON with `jq -n` instead of interpolating raw shell strings.

## Tools

Use the `bash` tool to call the API with `curl`.

## Fast Path

For common intents, map user requests like this:

- “看资产统计” -> `GET /api/v1/portfolio/stats`

### Curl base setup

```bash
BASE="${POLYHUB_API_BASE_URL%/}"
AUTH=(-H "Authorization: Bearer $POLYHUB_API_KEY" -H "Content-Type: application/json")
```

---

## Portfolio

### Action: Get portfolio stats

- `GET /api/v1/portfolio/stats`

Returns aggregated portfolio statistics for the authenticated user.

Current field semantics:

- `positionsValue`: official Polymarket positions value
- `availableBalance`: official USDC balance minus `unsettledFees`
- `totalPnL`: official Polymarket total PnL
- `unsettledFees`: unsettled Polyhub fees in USDC
- `investedCapital`: Polyhub-calculated invested capital for copy-task history

Current UI alignment in `poly_copy`:

- Portfolio page top stats use this endpoint
- Avatar dropdown `USDC Balance` uses `availableBalance`
- Avatar dropdown `Account Value` uses `availableBalance + positionsValue`

```bash
curl -sS --fail-with-body "${AUTH[@]}" "$BASE/api/v1/portfolio/stats"
```

---

## Error handling

- `401`: API key missing/invalid/expired/disabled. Ask user to check or regenerate key.
- `400`: Invalid request payload. Check required fields.
- `404`: Delegated access not registered for the given `organizationId`.
- `405`: Wrong HTTP method.
- `5xx`: Server error. Retry once with backoff; if still failing, report response body.
