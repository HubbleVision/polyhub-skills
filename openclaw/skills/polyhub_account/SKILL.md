---
name: polyhub_account
description: View portfolio stats, fee history and place manual orders on Polyhub using an API key.
---

# Polyhub Account Skill

Version: v0.3.0

## When to use

Use this skill when the user asks about:

- Portfolio overview or stats
- Fee records or fee history
- Placing a manual order on Polymarket

## Requirements

- `POLYHUB_API_BASE_URL` — Polyhub API server URL (e.g. `https://api.polyhub.example.com`)
- `POLYHUB_API_KEY` — API key (must start with `phub_`)
- `curl` must be available in the runtime environment.
- `jq` is strongly recommended for building JSON payloads safely.

## Safety rules

- Never print `POLYHUB_API_KEY` in the output.
- For `place-order` (a write action), repeat the full order summary and wait for explicit user confirmation before calling the API.
- Do not accept arbitrary JSON from the user for place-order. Ask for the minimal required fields one by one.
- Prefer building JSON with `jq -n` instead of interpolating raw shell strings.

## Tools

Use the `bash` tool to call the API with `curl`.

## Fast Path

For common intents, map user requests like this:

- “看资产统计” -> `GET /api/v1/portfolio/stats`
- “看手续费记录” -> `GET /api/v1/user/fees`
- “手动下单” -> `POST /api/v1/place-order`

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

Validation:

- `limit` must be a positive integer.
- `offset` must be a non-negative integer.

Example:

```bash
curl -sS --fail-with-body "${AUTH[@]}" "$BASE/api/v1/user/fees?limit=50&offset=0"
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
4. If `price` is omitted, prefer `isMarketOrder=true`.
5. Show a complete order summary and ask for confirmation.

Validation:

- `size` must be greater than `0`.
- `side` should be `BUY` or `SELL`.
- `tokenId`, `organizationId`, `signWith`, `safeAddress` must be non-empty.
- If delegated access is not registered, the API returns `404`.

Minimum fields to ask:

- Always ask: `tokenId`, `size`, `side`
- Always confirm: `organizationId`, `signWith`, `safeAddress`
- Always ask: `apiKey`, `apiSecret`, `apiPassphrase`
- Ask `price` only for limit orders
- Ask `negRisk` only if the user explicitly mentions a negative-risk market requirement

Decision rules:

- If the user does not specify a price, prefer a market order with `isMarketOrder=true`.
- If the user specifies a price target, use a limit order with `isMarketOrder=false` and include `price`.
- Use `size` as the primary amount field; only use `amount` when the caller explicitly wants that variant.
- Normalize `side` to uppercase before calling the API.

```bash
PAYLOAD="$(jq -n \
  --arg organizationId "..." \
  --arg signWith "..." \
  --arg safeAddress "0x..." \
  --arg tokenId "..." \
  --arg side "BUY" \
  --arg apiKey "..." \
  --arg apiSecret "..." \
  --arg apiPassphrase "..." \
  --argjson size 10 \
  --argjson isMarketOrder true \
  '{
    organizationId: $organizationId,
    signWith: $signWith,
    safeAddress: $safeAddress,
    tokenId: $tokenId,
    size: $size,
    side: $side,
    isMarketOrder: $isMarketOrder,
    apiKey: $apiKey,
    apiSecret: $apiSecret,
    apiPassphrase: $apiPassphrase
  }')"

curl -sS --fail-with-body "${AUTH[@]}" -X POST "$BASE/api/v1/place-order" \
  -d "$PAYLOAD"
```

Example: limit order

```bash
PAYLOAD="$(jq -n \
  --arg organizationId "..." \
  --arg signWith "..." \
  --arg safeAddress "0x..." \
  --arg tokenId "..." \
  --arg side "BUY" \
  --arg apiKey "..." \
  --arg apiSecret "..." \
  --arg apiPassphrase "..." \
  --argjson size 10 \
  --argjson price 0.57 \
  --argjson isMarketOrder false \
  '{
    organizationId: $organizationId,
    signWith: $signWith,
    safeAddress: $safeAddress,
    tokenId: $tokenId,
    size: $size,
    price: $price,
    side: $side,
    isMarketOrder: $isMarketOrder,
    apiKey: $apiKey,
    apiSecret: $apiSecret,
    apiPassphrase: $apiPassphrase
  }')"

curl -sS --fail-with-body "${AUTH[@]}" -X POST "$BASE/api/v1/place-order" \
  -d "$PAYLOAD"
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
- `405`: Wrong HTTP method.
- `5xx`: Server error. Retry once with backoff; if still failing, report response body.
