# 鉴权前置（Open API）

先按 [setup.md](setup.md) 在约定位置准备 `.env`。

问 mint / 写 `/v1/*` 集成时，**必须先完成**本页；再进入 Easy Mint 或 Deploy → Sponsor → Mint。

## 1. 读取凭证

从 [setup.md](setup.md) 约定路径的 `.env` 读取：

- `NFTRAINBOW_APP_ID`
- `NFTRAINBOW_APP_SECRET`
- `NFTRAINBOW_BASE_URL`（未填写时使用 `https://api.nftrainbow.cn`）

文件不存在或凭证为空时回到 `setup.md`；凭证就绪后进入 §2。

## 2. Login 换 JWT

Agent 按 §1 读取凭证后调用：

`POST {NFTRAINBOW_BASE_URL}/v1/login`（默认 Base 为 `https://api.nftrainbow.cn`）

Body：`{"app_id":"...","app_secret":"..."}`

```bash
ENV_FILE=".env" # Agent 替换为用户指定路径；未指定时保持默认值
set -a
. "$ENV_FILE"
set +a
: "${NFTRAINBOW_BASE_URL:=https://api.nftrainbow.cn}"

TOKEN=$(curl -fsS -X POST "$NFTRAINBOW_BASE_URL/v1/login" \
  -H 'Content-Type: application/json' \
  -d "{\"app_id\":\"$NFTRAINBOW_APP_ID\",\"app_secret\":\"$NFTRAINBOW_APP_SECRET\"}" \
  | python3 -c 'import sys,json; r=json.load(sys.stdin); print((r.get("data") or r)["token"])')
```

从响应取 `token`（可能在 `data.token` 或顶层 `token`），写入 shell 变量 `TOKEN`。

后续示例默认已设置好 `TOKEN` 与 `NFTRAINBOW_BASE_URL`（以及其余 `NFTRAINBOW_*`）。

**鉴权完成条件**：成功取得 JWT。

## 3. 调用时携带 Bearer

后续 `/v1/*` 请求头：

`Authorization: Bearer <token>`

## 4. 刷新（可选）

- `GET /v1/refresh_token`（Header 带当前 Bearer）
- JWT 约 **1 小时**内可调 Open API；约 **5 小时**内可 refresh；再过期则重新 login

## 官方文档

| 主题 | URL |
|------|-----|
| Authentication | https://docs.nftrainbow.xyz/about-the-apis/authentication |
| Login | https://docs.nftrainbow.xyz/api-reference/open-api/login |

## Agent 行为

- 用户问 login / JWT / 凭证 → 先按 [setup.md](setup.md) 准备 `.env`，再继续本页
- 约定文件中凭证就绪 → Agent 直接 login；不要求用户取得 JWT
