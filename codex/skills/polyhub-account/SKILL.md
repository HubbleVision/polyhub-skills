---
name: polyhub-account
description: View Polyhub portfolio stats, fee history, and place manual orders with explicit confirmation and field validation.
---

# Polyhub Account

Use this skill when the user wants account-level data or manual trading actions on Polyhub.

## Requirements

- `POLYHUB_API_BASE_URL` is set.
- `POLYHUB_API_KEY` is set and starts with `phub_`.
- `curl` is available.
- `jq` is recommended.

## Safety Rules

- Never print `POLYHUB_API_KEY`.
- For `place-order`, repeat the full order summary and wait for explicit confirmation before calling the API.
- Do not accept arbitrary JSON for order placement. Ask for minimal required fields first.
- Prefer `jq -n` for payload construction.

## Base Setup

```bash
BASE="${POLYHUB_API_BASE_URL%/}"
AUTH=(-H "Authorization: Bearer $POLYHUB_API_KEY" -H "Content-Type: application/json")
```

## Intent Mapping

- Portfolio overview: `GET /api/v1/portfolio/stats`
- Fee history: `GET /api/v1/user/fees`
- Manual order placement: `POST /api/v1/place-order`

## Common Calls

### Portfolio stats

```bash
curl -sS --fail-with-body "${AUTH[@]}" \
  "$BASE/api/v1/portfolio/stats"
```

### Fee history

Validation:

- `limit` must be positive.
- `offset` must be zero or greater.

```bash
curl -sS --fail-with-body "${AUTH[@]}" \
  "$BASE/api/v1/user/fees?limit=20&offset=0"
```

### Place order

Always ask for:

- `tokenId`
- `size`
- `side`
- `apiKey`
- `apiSecret`
- `apiPassphrase`

Always confirm:

- `organizationId`
- `signWith`
- `safeAddress`

Decision rules:

- If the user does not specify a price, prefer `isMarketOrder=true`.
- If the user specifies a target price, send a limit order with `price` and `isMarketOrder=false`.
- Normalize `side` to uppercase.

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

curl -sS --fail-with-body "${AUTH[@]}" \
  -X POST "$BASE/api/v1/place-order" \
  -d "$PAYLOAD"
```

## Error Handling

- `400`: invalid payload
- `401`: missing or invalid API key
- `404`: delegated access not registered
- `5xx`: server error
