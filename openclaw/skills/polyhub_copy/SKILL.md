---
name: polyhub_copy
description: Manage copy-trading tasks, view signals, positions and trades on Polyhub using an API key.
---

# Polyhub Copy Skill

Version: v0.1.0

## When to use

Use this skill when the user asks about:

- Listing, creating, updating or deleting copy-trading tasks
- Viewing copy task positions or trades
- Selling positions (single or all)
- Batch updating or deleting copy tasks
- Viewing total PnL across copy tasks
- Viewing copy signals and signal stats
- Managing take-profit / stop-loss (TPSL) rules

## Requirements

- `POLYHUB_API_BASE_URL` — Polyhub API server URL (e.g. `https://api.polyhub.example.com`)
- `POLYHUB_API_KEY` — API key (must start with `phub_`)

## Safety rules

- Never print `POLYHUB_API_KEY` in the output.
- Treat `taskId` as untrusted input. Validate format before calling the API.
- For write actions (`POST`/`PATCH`/`DELETE`), repeat the action summary and wait for explicit user confirmation before calling the API.
- Validation rules:
  - `taskId` must be a 24-char hex MongoDB ObjectID: `^[0-9a-fA-F]{24}$`

## Tools

Use the `bash` tool to call the API with `curl`.

### Curl base setup

```bash
BASE="${POLYHUB_API_BASE_URL%/}"
AUTH=(-H "Authorization: Bearer $POLYHUB_API_KEY" -H "Content-Type: application/json")
```

### Validate taskId helper

```bash
if [[ ! "$TASK_ID" =~ ^[0-9a-fA-F]{24}$ ]]; then echo "Invalid taskId"; exit 2; fi
```

---

## Copy Tasks

### Action: List copy tasks

- `GET /api/v1/copy-tasks`
- Query params: `includeDeleted=true|false` (default: true)

```bash
curl -sS --fail-with-body "${AUTH[@]}" "$BASE/api/v1/copy-tasks?includeDeleted=true"
```

### Action: Create copy task

- `POST /api/v1/copy-tasks`
- Required field: `targetTrader` (Polymarket address of the smart money)
- Optional fields: `targetUsername`, `taskConfig`, `filteredByTag`, `tpslRules`

Before calling: ask the user for `targetTrader` at minimum. Summarize and confirm.

```bash
curl -sS --fail-with-body "${AUTH[@]}" -X POST "$BASE/api/v1/copy-tasks" \
  -d '{"targetTrader":"0x..."}'
```

### Action: Get copy task

- `GET /api/v1/copy-tasks/{taskId}`

```bash
curl -sS --fail-with-body "${AUTH[@]}" "$BASE/api/v1/copy-tasks/$TASK_ID"
```

### Action: Update copy task

- `PATCH /api/v1/copy-tasks/{taskId}`
- Body: partial JSON with fields to update (e.g. `status`, `taskConfig`, `tpslRules`)

Before calling: validate `taskId`, summarize changes, and confirm.

```bash
curl -sS --fail-with-body "${AUTH[@]}" -X PATCH "$BASE/api/v1/copy-tasks/$TASK_ID" \
  -d "$BODY"
```

### Action: Delete copy task

- `DELETE /api/v1/copy-tasks/{taskId}`

Before calling: validate `taskId` and confirm.

```bash
curl -sS --fail-with-body "${AUTH[@]}" -X DELETE "$BASE/api/v1/copy-tasks/$TASK_ID"
```

### Action: Get total PnL

- `GET /api/v1/copy-tasks/total-pnl`

```bash
curl -sS --fail-with-body "${AUTH[@]}" "$BASE/api/v1/copy-tasks/total-pnl"
```

---

## Positions & Trades

### Action: List positions

- `GET /api/v1/copy-tasks/{taskId}/positions`
- Query params: `status` (optional, e.g. `open`, `closed`)

```bash
curl -sS --fail-with-body "${AUTH[@]}" "$BASE/api/v1/copy-tasks/$TASK_ID/positions?status=open"
```

### Action: List trades

- `GET /api/v1/copy-tasks/{taskId}/trades`

```bash
curl -sS --fail-with-body "${AUTH[@]}" "$BASE/api/v1/copy-tasks/$TASK_ID/trades"
```

### Action: Sell position

- `POST /api/v1/copy-tasks/{taskId}/sell`
- Required field: `marketId` (or `conditionId`)
- Optional field: `amount` (partial sell; omit to sell full position)

Before calling: validate `taskId`, summarize (which market, amount), and confirm.

```bash
curl -sS --fail-with-body "${AUTH[@]}" -X POST "$BASE/api/v1/copy-tasks/$TASK_ID/sell" \
  -d '{"marketId":"...","amount":10}'
```

### Action: Sell all positions

- `POST /api/v1/copy-tasks/{taskId}/sell-all`

Before calling: validate `taskId` and confirm — this sells ALL positions for the task.

```bash
curl -sS --fail-with-body "${AUTH[@]}" -X POST "$BASE/api/v1/copy-tasks/$TASK_ID/sell-all"
```

---

## Batch Operations

### Action: Batch update tasks

- `POST /api/v1/copy-tasks/batch-update`
- Required: `taskIds` (array of task IDs)
- Optional: `status` (string), `taskConfig` (object)

Before calling: summarize which tasks and what changes, and confirm.

```bash
curl -sS --fail-with-body "${AUTH[@]}" -X POST "$BASE/api/v1/copy-tasks/batch-update" \
  -d '{"taskIds":["id1","id2"],"status":"paused"}'
```

### Action: Batch delete tasks

- `POST /api/v1/copy-tasks/batch-delete`
- Required: `taskIds` (array of task IDs)

Before calling: confirm deletion.

```bash
curl -sS --fail-with-body "${AUTH[@]}" -X POST "$BASE/api/v1/copy-tasks/batch-delete" \
  -d '{"taskIds":["id1","id2"]}'
```

---

## Copy Signals

### Action: List copy signals

- `GET /api/v1/copy-signals`
- Query params:
  - `limit` (int, default 50)
  - `cursor` (int, offset for pagination)
  - `since` (RFC3339 timestamp)
  - `sinceCreatedAt` (RFC3339 timestamp)
  - `action` (one of: `COPIED`, `SKIPPED`, `FAILED`, `RECEIVED`)
  - `trader` (address filter)

```bash
curl -sS --fail-with-body "${AUTH[@]}" "$BASE/api/v1/copy-signals?limit=20&action=COPIED"
```

### Action: Get signal stats

- `GET /api/v1/copy-signals/stats`

Returns signal counts grouped by action and by trader.

```bash
curl -sS --fail-with-body "${AUTH[@]}" "$BASE/api/v1/copy-signals/stats"
```

---

## TPSL Rules (Take-Profit / Stop-Loss)

### Action: Get TPSL rule

- `GET /api/v1/copy-tasks/{taskId}/tpsl-rules/{ruleId}`

```bash
curl -sS --fail-with-body "${AUTH[@]}" "$BASE/api/v1/copy-tasks/$TASK_ID/tpsl-rules/$RULE_ID"
```

### Action: Simulate TPSL rule

- `POST /api/v1/copy-tasks/{taskId}/tpsl-rules/{ruleId}/simulate`

```bash
curl -sS --fail-with-body "${AUTH[@]}" -X POST "$BASE/api/v1/copy-tasks/$TASK_ID/tpsl-rules/$RULE_ID/simulate"
```

---

## Error handling

- `401`: API key missing/invalid/expired/disabled. Ask user to check or regenerate key.
- `404`: Task or resource not found. Ask user to verify `taskId`.
- `409`: Task already exists (duplicate `targetTrader`).
- `422`: Copy task limit exceeded.
- `5xx`: Server error. Retry once with backoff; if still failing, report response body.
