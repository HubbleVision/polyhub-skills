---
name: polyhub_account
description: View portfolio stats, fee history and place manual orders on Polyhub using an API key.
---

# Polyhub Account Skill

Version: v0.1.0

## When to use

Use this skill when the user asks about:

- Portfolio overview or stats
- Fee records or fee history
- Placing a manual order on Polymarket

## Requirements

- `POLYHUB_API_BASE_URL` — Polyhub API server URL (e.g. `https://api.polyhub.example.com`)
- `POLYHUB_API_KEY` — API key (must start with `phub_`)

## Safety rules

- Never print `POLYHUB_API_KEY` in the output.
- For `place-order` (a write action), repeat the full order summary and wait for explicit user confirmation before calling the API.
- Do not accept arbitrary JSON from the user for place-order. Ask for the minimal required fields one by one.

## Tools

Use the `bash` tool to call the API with `curl`.

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

```bash
curl -sS --fail-with-body "${AUTH[@]}" "$BASE/api/v1/portfolio/stats"
```

---

## Fees

### Action: List fee records

- `GET /api/v1/user/fees`
- Query params:
  - `limit` (int, default 20)
  - `offset` (int, default 0)

```bash
curl -sS --fail-with-body "${AUTH[@]}" "$BASE/api/v1/user/fees?limit=20&offset=0"
```

---

## Place Order

### Action: Place a manual order

- `POST /api/v1/place-order`

This places an order on Polymarket via delegated access. It requires a pre-configured delegated access setup (Turnkey organization).

Required fields:

| Field | Type | Description |
|-------|------|-------------|
| `organizationId` | string | Turnkey organization ID |
| `signWith` | string | Turnkey signing key ID |
| `safeAddress` | string | User's Safe (proxy wallet) address |
| `tokenId` | string | Polymarket condition token ID |
| `size` | number | Order size in USDC (must be > 0) |
| `side` | string | `BUY` or `SELL` |
| `apiKey` | string | CLOB API key |
| `apiSecret` | string | CLOB API secret |
| `apiPassphrase` | string | CLOB API passphrase |

Optional fields:

| Field | Type | Description |
|-------|------|-------------|
| `amount` | number | Alternative to `size` |
| `price` | number | Limit price (omit for market order) |
| `negRisk` | bool | Negative-risk market flag |
| `isMarketOrder` | bool | Force market order |

Before calling:

1. Ask the user for `tokenId`, `size`, `side` at minimum.
2. Confirm `organizationId`, `signWith`, `safeAddress` — these may come from user config.
3. CLOB credentials (`apiKey`, `apiSecret`, `apiPassphrase`) are required.
4. Show a complete order summary and ask for confirmation.

```bash
curl -sS --fail-with-body "${AUTH[@]}" -X POST "$BASE/api/v1/place-order" \
  -d '{
    "organizationId": "...",
    "signWith": "...",
    "safeAddress": "0x...",
    "tokenId": "...",
    "size": 10,
    "side": "BUY",
    "isMarketOrder": true,
    "apiKey": "...",
    "apiSecret": "...",
    "apiPassphrase": "..."
  }'
```

Response on success:

```json
{"orderId": "0x..."}
```

---

## Error handling

- `401`: API key missing/invalid/expired/disabled. Ask user to check or regenerate key.
- `400`: Invalid request payload. Check required fields.
- `404`: Delegated access not registered for the given `organizationId`.
- `5xx`: Server error. Retry once with backoff; if still failing, report response body.
