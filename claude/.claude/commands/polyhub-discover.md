# /polyhub-discover

Use this command for Polyhub public discover queries.

Interpret `$ARGUMENTS` as the user's discover request. Use the terminal and `curl` to call the Polyhub public API.

## Requirements

- `POLYHUB_API_BASE_URL` is set
- `curl` is available
- `POLYHUB_API_KEY` is not required

## Base Setup

```bash
BASE="${POLYHUB_API_BASE_URL%/}"
```

## Intent Mapping

- Discover tags: `GET /api/v1/markets/tags`
- Trader rankings by tag: `GET /api/v1/traders-v2/`
- Trader detail by address: `GET /api/v1/traders/by-address`
- Market tag lookup by condition IDs: `GET /api/v1/markets/by-condition-ids`

## Rules

- These endpoints are read-only; no confirmation step is required.
- Do not invent filters or sort values.
- Keep wallet address casing if the user provided it.
- Summarize large responses instead of dumping raw JSON.

## Examples

### List tags

```bash
curl -sS --fail-with-body "$BASE/api/v1/markets/tags"
```

### Query traders for a tag

```bash
curl -sS --fail-with-body \
  "$BASE/api/v1/traders-v2/?tag=Politics&time_range=30d&limit=10&offset=0"
```

### Trader by address

```bash
curl -sS --fail-with-body \
  "$BASE/api/v1/traders/by-address?user_id=0x1234..."
```

### Condition ID lookup

```bash
curl -sS --fail-with-body \
  "$BASE/api/v1/markets/by-condition-ids?ids=0xabc,0xdef"
```
