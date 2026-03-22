# Codex 使用说明（polyhub-skills）

本仓库提供一组可直接被 Codex 加载的 Skills，目录位于：

- `codex/skills/`

当前版本：`v0.3.6`

## 包含的 Skills

| Skill | 说明 |
|-------|------|
| `polyhub-discover` | Discover 页面公开查询：标签列表、交易员排行、跨标签查询、地址详情、condition id 标签映射 |
| `polyhub-copy` | 跟单任务管理：创建/更新/删除跟单任务、查看持仓/交易、卖出、批量操作、信号监控、TPSL 规则 |
| `polyhub-account` | 账户概览：投资组合统计、费用记录、手动下单 |

## 前置条件

- 已安装 Codex
- 如果要使用 `polyhub-copy` / `polyhub-account`，还需要可用的 Polyhub API Key（前缀为 `phub_`）

## 环境变量

### `polyhub-discover`

```bash
# 默认直接使用公开地址，无需配置环境变量
# https://polyhub.skill-test.bedev.hubble-rpc.xyz
```

如果是自定义部署，也可以覆盖：

```bash
export POLYHUB_API_BASE_URL="https://api.polyhub.example.com"
```

### `polyhub-copy` / `polyhub-account` 必需

```bash
export POLYHUB_API_BASE_URL="https://api.polyhub.example.com"
export POLYHUB_API_KEY="phub_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```

## 安装 Skills

Codex 默认从 `~/.codex/skills/` 加载 skills。

如果目录不存在，先创建：

```bash
mkdir -p ~/.codex/skills
```

### 方案 A：symlink（推荐）

```bash
ln -sfn "$(pwd)/codex/skills/polyhub-discover" ~/.codex/skills/polyhub-discover
ln -sfn "$(pwd)/codex/skills/polyhub-copy" ~/.codex/skills/polyhub-copy
ln -sfn "$(pwd)/codex/skills/polyhub-account" ~/.codex/skills/polyhub-account
```

### 方案 B：copy

```bash
cp -R codex/skills/* ~/.codex/skills/
```

## 验证

- 进入 Codex 后，让它执行 discover 查询，例如“列出 Polyhub discover tags”
- 如果使用鉴权 skill，再让它执行一次“列出 copy tasks”或“查看 portfolio stats”

## 安全建议

- 不要把 `POLYHUB_API_KEY` 提交到 Git
- 不要把 `POLYHUB_API_KEY` 打印到日志、截图或聊天记录
- 对写操作保持二次确认，尤其是 `place-order`、`PATCH`、`DELETE`、`sell`、`sell-all`
