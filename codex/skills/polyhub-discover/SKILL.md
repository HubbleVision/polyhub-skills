---
name: polyhub-discover
description: Query Polyhub public discover APIs for tags, trader rankings, trader detail by address, and market tag lookup by condition IDs.
---

# Polyhub Discover

Version: v0.4.1

Use this skill when the user wants public discover data from Polyhub.

## Requirements

- `curl` is available.
- `POLYHUB_API_KEY` is not required.

`POLYHUB_API_BASE_URL` is fixed to `https://polyhub.skill-test.bedev.hubble-rpc.xyz`.

## Workflow

1. Use the terminal to call Polyhub public APIs with `curl`.
2. If the user provides explicit filters or a full query shape, preserve them exactly.
3. Do not silently change `tag`, `sort_by`, `sort_direction`, numeric thresholds, or add extra filters.
4. Summarize results clearly instead of dumping raw JSON when the response is large.

## Base Setup

```bash
BASE="https://polyhub.skill-test.bedev.hubble-rpc.xyz"
```

## Intent Mapping

- List discover tags: `GET /api/v1/markets/tags`
- List traders by tag: `GET /api/v1/traders-v2/`
- Look up trader stats by address: `GET /api/v1/traders/by-address`
- Look up market tags by condition IDs: `GET /api/v1/markets/by-condition-ids`

## Common Calls

### List tags

```bash
curl -sS --fail-with-body "$BASE/api/v1/markets/tags"
```

### List traders for a tag

Priority:

1. If the user gives explicit filters, map them 1:1 into query params.
2. If the user references a visible Discover page request, mirror that request instead of using skill defaults.
3. Only use the defaults below when the user did not provide enough detail.

Required params:

- `tag`
- `time_range`: `all` or `30d`

Optional params:

- `limit`
- `offset`
- `filterBots`
- `sort_by`
- `sort_direction`
- numeric range filters such as `pnl_min`, `roi_min`, `trade_count_30_min`

```bash
curl -sS --fail-with-body \
  "$BASE/api/v1/traders-v2/?tag={USER_TAG_OR_Politics}&time_range=30d&filterBots=1&pnl_min=1000&trade_count_30_min=30&trade_count_30_max=3000&sort_by=timing_score&sort_direction=desc&limit=20&offset=0"
```

Guidance:

- Use `tag=CROSS-TAG` when the user wants cross-tag discover results.
- Use `time_range=30d` for recent performance.
- Prefer descending sort unless the user asked otherwise.
- Preserve backend row order unless the user explicitly asks for re-ranking.
- Keep `tag` user-controlled. If the user specifies a tag, use it directly; otherwise default to `Politics` to match the webpage.

Default Discover query shape when the user asks for the standard webpage-style leaderboard without extra detail:

```bash
curl -sS --fail-with-body \
  "$BASE/api/v1/traders-v2/?tag={USER_TAG_OR_Politics}&limit=20&offset=0&time_range=30d&filterBots=1&pnl_min=1000&trade_count_30_min=30&trade_count_30_max=3000&sort_by=timing_score&sort_direction=desc"
```

### Trader detail by address

```bash
curl -sS --fail-with-body \
  "$BASE/api/v1/traders/by-address?user_id=0x1234..."
```

### Market tag lookup by condition IDs

Validation:

- `ids` must be non-empty.
- Limit to `200` IDs per request.

```bash
curl -sS --fail-with-body \
  "$BASE/api/v1/markets/by-condition-ids?ids=0xabc,0xdef"
```

## Error Handling

- `400`: invalid params
- `500`: backend query failed or unavailable
