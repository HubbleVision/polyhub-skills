---
name: polyhub_discover
description: Explore public discover data on Polyhub without API key auth, including tags, trader rankings, trader detail stats, and market tag lookup.
---

# Polyhub Discover Skill

Version: v0.3.6

## When to use

Use this skill when the user asks about:

- Discover page tag list
- Trader rankings on the discover page
- Cross-tag discover queries
- Filtering and sorting discover traders
- Looking up a trader by address
- Looking up market tags by condition IDs

## Requirements

- `curl` must be available in the runtime environment

`POLYHUB_API_BASE_URL` is optional. If it is not set, default to `https://polyhub.skill-test.bedev.hubble-rpc.xyz`.

This skill does not require `POLYHUB_API_KEY`.

## Safety rules

- These are public read-only endpoints. No confirmation step is required.
- Do not invent filter values. Only pass filters the user requested.
- Prefer building query strings from explicit user intent.
- When querying by address, trim whitespace and keep the original checksum/casing if provided.

## Tools

Use the `bash` tool to call the API with `curl`.

## Fast Path

For common intents, map user requests like this:

- “discover 页有哪些标签” -> `GET /api/v1/markets/tags`
- “看某个 tag 的 trader 排行” -> `GET /api/v1/traders-v2/?tag=...`
- “跨 tag 找高手” -> `GET /api/v1/traders-v2/?tag=CROSS-TAG`
- “看某个地址在各标签下的数据” -> `GET /api/v1/traders/by-address?user_id=...`
- “查 condition id 对应什么标签” -> `GET /api/v1/markets/by-condition-ids?ids=...`

### Curl base setup

```bash
BASE="${POLYHUB_API_BASE_URL:-https://polyhub.skill-test.bedev.hubble-rpc.xyz}"
BASE="${BASE%/}"
```

---

## Tags

### Action: List discover tags

- `GET /api/v1/markets/tags`
- Auth: public

```bash
curl -sS --fail-with-body "$BASE/api/v1/markets/tags"
```

Use this when the user wants the discover page tag list or wants to browse available niches first.

---

## Trader Rankings

### Action: List traders for discover

- `GET /api/v1/traders-v2/`
- Auth: public

Core query params:

- `tag` — required
- `time_range` — required: `all` or `30d`
- `limit` — optional, default `10`, max `100`
- `offset` — optional, default `0`
- `filterBots` — optional: `0` or `1`
- `sort_by` — optional: `volume`, `pnl`, `roi`, `avg_adt`, `trade_count_30`, `ev_per_bought`, `timing_score`
- `sort_direction` — optional: `asc` or `desc`

Range filter params:

- `pnl_min`, `pnl_max`
- `volume_min`, `volume_max`
- `roi_min`, `roi_max`
- `avg_adt_min`, `avg_adt_max`
- `trade_count_30_min`, `trade_count_30_max`
- `ev_per_bought_min`, `ev_per_bought_max`
- `timing_score_min`, `timing_score_max`

Validation:

- `tag` is required
- `time_range` must be `all` or `30d`
- `limit` should be between `1` and `100`
- `offset` should be `0` or greater
- `filterBots` should be `0` or `1`

Example: standard discover query

```bash
curl -sS --fail-with-body \
  "$BASE/api/v1/traders-v2/?tag=Politics&time_range=all&limit=10&offset=0"
```

Example: cross-tag query

```bash
curl -sS --fail-with-body \
  "$BASE/api/v1/traders-v2/?tag=CROSS-TAG&time_range=30d&limit=20&offset=0"
```

Example: filtered and sorted query

```bash
curl -sS --fail-with-body \
  "$BASE/api/v1/traders-v2/?tag=Sports&time_range=30d&filterBots=1&pnl_min=1000&trade_count_30_min=30&sort_by=ev_per_bought&sort_direction=desc"
```

Guidance:

- Use `tag=CROSS-TAG` when the user wants discover results across all tags.
- Use `time_range=30d` when the user asks for recent performance.
- Use `filterBots=1` when the user explicitly wants bot filtering.
- Prefer `sort_direction=desc` unless the user explicitly wants ascending order.

---

## Trader Detail

### Action: Get trader stats by address

- `GET /api/v1/traders/by-address`
- Auth: public

Required query params:

- `user_id` — trader wallet address

Optional query params:

- `time_range` — if supported by caller flow

```bash
curl -sS --fail-with-body \
  "$BASE/api/v1/traders/by-address?user_id=0x1234..."
```

Use this when the user clicks into a trader from discover and wants stats across tags.

---

## Market Tag Lookup

### Action: Get market tags by condition IDs

- `GET /api/v1/markets/by-condition-ids`
- Auth: public

Required query params:

- `ids` — comma-separated condition ID list

Validation:

- `ids` must not be empty
- The backend supports up to `200` IDs per request

```bash
curl -sS --fail-with-body \
  "$BASE/api/v1/markets/by-condition-ids?ids=0xabc,0xdef"
```

Use this when the user wants to map market condition IDs back to discover tags.

---

## Error handling

- `400`: Invalid query parameters such as missing `tag`, invalid `time_range`, or empty `ids`
- `500`: Backend query failed or service unavailable
