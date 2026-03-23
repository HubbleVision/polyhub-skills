# /polyhub-copy

Version: v0.3.7

Use this command for Polyhub copy-trading task management.

Interpret `$ARGUMENTS` as the user's copy-trading intent. Use the terminal and `curl` to call the Polyhub authenticated API.

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
- Treat all IDs and JSON-like user input as untrusted.
- For write actions, restate the action and wait for explicit confirmation.
- Prefer `jq -n` to build payloads.
- Validate `taskId` with `^[0-9a-fA-F]{24}$`.

## Intent Mapping

- List tasks: `GET /api/v1/copy-tasks`
- Create task: `POST /api/v1/copy-tasks`
- Update task: `PATCH /api/v1/copy-tasks/{taskId}`
- Inspect task: `GET /api/v1/copy-tasks/{taskId}`
- Inspect signals: `GET /api/v1/copy-signals`
- Inspect signal stats: `GET /api/v1/copy-signals/stats`

## Examples

### List tasks

```bash
curl -sS --fail-with-body "${AUTH[@]}" \
  "$BASE/api/v1/copy-tasks?includeDeleted=true"
```

### Create task

```bash
PAYLOAD="$(jq -n --arg targetTrader "0x..." '{targetTrader: $targetTrader}')"

curl -sS --fail-with-body "${AUTH[@]}" \
  -X POST "$BASE/api/v1/copy-tasks" \
  -d "$PAYLOAD"
```

### Pause task

```bash
PAYLOAD="$(jq -n --arg status "PAUSED" '{status: $status}')"

curl -sS --fail-with-body "${AUTH[@]}" \
  -X PATCH "$BASE/api/v1/copy-tasks/$TASK_ID" \
  -d "$PAYLOAD"
```
