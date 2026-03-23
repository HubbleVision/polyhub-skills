# /polyhub-account

Version: v0.3.7

Use this command for Polyhub account queries and manual order placement.

Interpret `$ARGUMENTS` as the user's account or order intent. Use the terminal and `curl` to call the Polyhub authenticated API.

## Requirements

- `POLYHUB_API_BASE_URL` is set
- `POLYHUB_API_KEY` is set and starts with `phub_`
- `curl` is available
- `jq` is recommended

## Base Setup

```bash
BASE="${POLYHUB_API_BASE_URL%/}"
AUTH=(-H "Authorization: Bearer $POLYHUB_API_KEY" -H "Content-Type: application/json")
```

## Rules

- Never print `POLYHUB_API_KEY`.
- For `place-order`, repeat the full order summary and require explicit confirmation.
- Do not accept arbitrary JSON payloads for order placement.
- Prefer `jq -n` to build payloads.

## Intent Mapping

- Portfolio stats: `GET /api/v1/portfolio/stats`
- Fee history: `GET /api/v1/user/fees`
- Manual order: `POST /api/v1/place-order`

## Examples

### Portfolio stats

Field semantics:

- `positionsValue`: official Polymarket positions value
- `availableBalance`: official USDC balance minus `unsettledFees`
- `totalPnL`: official Polymarket total PnL
- `unsettledFees`: unsettled Polyhub fees in USDC
- `investedCapital`: Polyhub-calculated invested capital for copy-task history

UI alignment:

- `poly_copy` portfolio header uses this endpoint
- avatar dropdown `USDC Balance` uses `availableBalance`
- avatar dropdown `Account Value` uses `availableBalance + positionsValue`

```bash
curl -sS --fail-with-body "${AUTH[@]}" \
  "$BASE/api/v1/portfolio/stats"
```

### Fee history

```bash
curl -sS --fail-with-body "${AUTH[@]}" \
  "$BASE/api/v1/user/fees?limit=20&offset=0"
```

### Market order

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
