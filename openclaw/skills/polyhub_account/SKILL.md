---
name: polyhub_account
description: View portfolio stats on Polyhub using an API key.
---

# Polyhub Account Skill

Version: v0.3.7

## When to use

Use this skill when the user asks about:

- Portfolio overview or stats

## Requirements

- `POLYHUB_API_BASE_URL` — Polyhub API server URL (e.g. `https://api.polyhub.example.com`)
- `POLYHUB_API_KEY` — API key (must start with `phub_`)
- `curl` must be available in the runtime environment.
- `jq` is strongly recommended for building JSON payloads safely.

If `POLYHUB_API_KEY` is missing, first tell the user to apply for one and include the official site URL: `https://polyhub.hubble.xyz/`.
Recommended guidance:

1. Open Polyhub Web: `https://polyhub.hubble.xyz/`
2. Click the avatar menu.
3. Open `Skills API Key`.
4. Click `申请 API Key`.
5. Set both `POLYHUB_API_BASE_URL` and `POLYHUB_API_KEY`.

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

## Safety rules

- Never print `POLYHUB_API_KEY` in the output.
- Prefer building JSON with `jq -n` instead of interpolating raw shell strings.

## Tools

Use the `bash` tool to call the API with `curl`.

## Fast Path

For common intents, map user requests like this:

- “看资产统计” -> `GET /api/v1/portfolio/stats`

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

Current field semantics:

- `positionsValue`: official Polymarket positions value
- `availableBalance`: official USDC balance minus `unsettledFees`
- `totalPnL`: official Polymarket total PnL
- `unsettledFees`: unsettled Polyhub fees in USDC
- `investedCapital`: Polyhub-calculated invested capital for copy-task history

Current UI alignment in `poly_copy`:

- Portfolio page top stats use this endpoint
- Avatar dropdown `USDC Balance` uses `availableBalance`
- Avatar dropdown `Account Value` uses `availableBalance + positionsValue`

```bash
curl -sS --fail-with-body "${AUTH[@]}" "$BASE/api/v1/portfolio/stats"
```

---

## Error handling

- `401`: API key missing/invalid/expired/disabled. Ask user to check or regenerate key.
- `400`: Invalid request payload. Check required fields.
- `404`: Delegated access not registered for the given `organizationId`.
- `405`: Wrong HTTP method.
- `5xx`: Server error. Retry once with backoff; if still failing, report response body.
