---
name: polyhub-copy
description: Manage Polyhub copy-trading tasks, positions, trades, signals, sell flows, batch operations, and TPSL rules with an API key.
---

# Polyhub Copy

Version: v0.3.7

Use this skill when the user wants to manage copy-trading tasks on Polyhub.

## Requirements

- `POLYHUB_API_BASE_URL` is set.
- `POLYHUB_API_KEY` is set and starts with `phub_`.
- `curl` is available.
- `jq` is recommended.

If `POLYHUB_API_KEY` is missing, tell the user they need to apply for one first and include the official site URL: `https://polyhub.hubble.xyz/`.
Recommended guidance:

1. Open Polyhub Web: `https://polyhub.hubble.xyz/`
2. Click the avatar menu.
3. Open `Skills API Key`.
4. Click `申请 API Key`.
5. Set:
   - `POLYHUB_API_BASE_URL`
   - `POLYHUB_API_KEY`

Suggested wording:

```text
还没配置 API Key。要使用这个 Polyhub 鉴权功能，需要：
1. POLYHUB_API_BASE_URL：你的 Polyhub API 地址
2. POLYHUB_API_KEY：你的 API Key（以 phub_ 开头）

申请地址：
https://polyhub.hubble.xyz/

获取方式：
打开 Polyhub 官网 -> 点击头像 -> Skills API Key -> 申请 API Key

配置好之后直接发给我，或者先在环境变量里设置好，我再继续帮你处理。
```

## Safety Rules

- Never print `POLYHUB_API_KEY`.
- Treat IDs and user-provided JSON-like input as untrusted.
- For write actions, restate the full action summary and wait for explicit confirmation before calling the API.
- Prefer `jq -n` instead of interpolating raw JSON.
- Validate `taskId` with `^[0-9a-fA-F]{24}$`.

## Base Setup

```bash
BASE="${POLYHUB_API_BASE_URL%/}"
AUTH=(-H "Authorization: Bearer $POLYHUB_API_KEY" -H "Content-Type: application/json")
```

## Intent Mapping

- List tasks: `GET /api/v1/copy-tasks`
- Get one task: `GET /api/v1/copy-tasks/{taskId}`
- Create task: `POST /api/v1/copy-tasks`
- Update task: `PATCH /api/v1/copy-tasks/{taskId}`
- View signals: `GET /api/v1/copy-signals`
- View signal stats: `GET /api/v1/copy-signals/stats`
- View positions or trades: use the task-specific positions or trades endpoints requested by the user
- Sell one position or all positions: use `sell` or `sell-all` flows after confirmation

## Validation Helpers

```bash
if [[ ! "$TASK_ID" =~ ^[0-9a-fA-F]{24}$ ]]; then
  echo "Invalid taskId"
  exit 2
fi
```

## Common Calls

### List copy tasks

```bash
curl -sS --fail-with-body "${AUTH[@]}" \
  "$BASE/api/v1/copy-tasks?includeDeleted=true"
```

Guidance:

- Use `includeDeleted=true` for history or deleted tasks.
- Use `includeDeleted=false` for active tasks only.

### Create copy task

Always ask for:

- `targetTrader`

Ask only when needed:

- `targetUsername`
- `filteredByTag`
- `taskConfig`
- `tpslRules`

```bash
PAYLOAD="$(jq -n --arg targetTrader "0x..." '{targetTrader: $targetTrader}')"

curl -sS --fail-with-body "${AUTH[@]}" \
  -X POST "$BASE/api/v1/copy-tasks" \
  -d "$PAYLOAD"
```

### Update copy task

Common fields:

- `status`
- `taskConfig`
- `filteredByTag`
- `targetUsername`
- `tpslRules`

```bash
PAYLOAD="$(jq -n --arg status "PAUSED" '{status: $status}')"

curl -sS --fail-with-body "${AUTH[@]}" \
  -X PATCH "$BASE/api/v1/copy-tasks/$TASK_ID" \
  -d "$PAYLOAD"
```

Copy mode guidance:

- `ONE_TO_ONE`
- `FIXED_SIZE`, which should include `taskConfig.fixedAmount`

### Example: switch to fixed-size copy

```bash
PAYLOAD="$(jq -n \
  --argjson fixedAmount 5 \
  '{taskConfig: {copyMode: "FIXED_SIZE", fixedAmount: $fixedAmount}}')"
```

## Error Handling

- `400`: invalid payload
- `401`: missing or invalid API key
- `404`: resource not found
- `5xx`: server error
