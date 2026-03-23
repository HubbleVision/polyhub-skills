# OpenClaw 使用说明（polyhub-skills）

本仓库提供一组可直接被 OpenClaw 加载的 Skills，目录位于：

- `openclaw/skills/`

当前版本：`v0.3.8`

## 前置条件

- 已安装并运行 OpenClaw（本机或服务器均可）
- 如果要使用 `polyhub_copy` / `polyhub_account`，还需要可用的 Polyhub API Key（前缀为 `phub_`）

## 环境变量

### `polyhub_discover`

默认直接使用公开地址：

```bash
https://polyhub.skill-test.bedev.hubble-rpc.xyz
```

固定 API 地址：

```bash
export POLYHUB_API_BASE_URL="https://polyhub.skill-test.bedev.hubble-rpc.xyz"
```

### `polyhub_copy` / `polyhub_account` 必需

- `POLYHUB_API_BASE_URL`
  - 固定为 `https://polyhub.skill-test.bedev.hubble-rpc.xyz`

- `POLYHUB_API_KEY`
  - 你的 API key，必须以 `phub_` 开头

示例：

```bash
export POLYHUB_API_BASE_URL="https://polyhub.skill-test.bedev.hubble-rpc.xyz"
export POLYHUB_API_KEY="phub_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```

安全建议：

- 不要把 `POLYHUB_API_KEY` 提交到 Git
- 不要把 `POLYHUB_API_KEY` 打印到日志/截图/聊天记录

## 包含的 Skills

| Skill | 说明 |
|-------|------|
| `polyhub_discover` | Discover 页面公开查询：标签列表、交易员排行、跨标签查询、地址详情、condition id 标签映射 |
| `polyhub_copy` | 跟单任务管理：创建/更新/删除跟单任务、查看持仓/交易、卖出、批量操作、信号监控、TPSL 规则 |
| `polyhub_account` | 账户概览：投资组合统计、费用记录、手动下单 |

## 安装 Skills（copy 或 symlink）

OpenClaw 通常从以下路径加载 Skills：

- `~/.openclaw/workspace/skills/`

如果目录不存在，先创建：

```bash
mkdir -p ~/.openclaw/workspace/skills
```

### 方案 A：symlink（推荐，方便升级）

```bash
ln -sfn "$(pwd)/openclaw/skills/polyhub_discover" ~/.openclaw/workspace/skills/polyhub_discover
ln -sfn "$(pwd)/openclaw/skills/polyhub_copy"    ~/.openclaw/workspace/skills/polyhub_copy
ln -sfn "$(pwd)/openclaw/skills/polyhub_account"  ~/.openclaw/workspace/skills/polyhub_account
```

### 方案 B：copy（适合"拷贝到服务器后离线使用"）

```bash
cp -R openclaw/skills/* ~/.openclaw/workspace/skills/
```

## 运行用户提示

Skills 和环境变量必须配置在"运行 OpenClaw 的那个用户"下：

- 如果 OpenClaw 以 `root` 运行，路径就是：`/root/.openclaw/workspace/skills/`
- 如果以普通用户运行，就是：`/home/<user>/.openclaw/workspace/skills/`

如果你不确定 OpenClaw 以谁运行：

```bash
ps -ef | grep -i openclaw | grep -v grep
```

## 验证（最小自检）

只要环境变量已生效、Skills 已被加载，你可以让 OpenClaw 使用 Skill 访问接口。

### 验证公开 discover skill

```bash
BASE="https://polyhub.skill-test.bedev.hubble-rpc.xyz"
curl -sS --fail-with-body "$BASE/api/v1/markets/tags"
```

### 验证需要 API key 的 skills

也可以直接用 `curl` 验证 API Key 是否正确（不依赖 OpenClaw）：

```bash
BASE="https://polyhub.skill-test.bedev.hubble-rpc.xyz"
curl -sS --fail-with-body \
  -H "Authorization: Bearer $POLYHUB_API_KEY" \
  -H "Content-Type: application/json" \
  "$BASE/api/v1/copy-tasks"
```

## 常见问题排查

- `401 Unauthorized`：`POLYHUB_API_KEY` 缺失/无效/过期/被禁用
- `404 Not Found`：URL 路径或 `taskId` 等参数不正确
- 公开 discover skill 访问失败：优先检查固定地址是否可达，以及目标服务是否暴露 `/api/v1/markets/tags`、`/api/v1/traders-v2/` 等公开接口
- Skills 不生效：优先确认"运行用户"是否一致，以及 OpenClaw 进程是否需要重启才会重新加载 Skills
