# 鉴权前置（Open API）

先完成 KYC（实名认证），见 [kyc.md](kyc.md)。

问 mint / 写 `/v1/*` 集成时，**必须先完成**本页；再进入 Easy Mint 或 Deploy → Sponsor → Mint。

## 1. 创建 App 并获取凭证

1. 注册并登录 [Rainbow 控制台](https://console.nftrainbow.cn)
2. 创建应用（App）
3. 取得 `app_id` 与 `app_secret`

环境变量写法见 [examples/env.md](../examples/env.md)。**禁止**把真实 secret 写入仓库。

## 2. Login 换 JWT

先按 §1 设置好 `NFTRAINBOW_BASE_URL` / `NFTRAINBOW_APP_ID` / `NFTRAINBOW_APP_SECRET`，再调用：

`POST {NFTRAINBOW_BASE_URL}/v1/login`（默认 Base 为 `https://api.nftrainbow.cn`）

Body：`{"app_id":"...","app_secret":"..."}`

```bash
TOKEN=$(curl -fsS -X POST "$NFTRAINBOW_BASE_URL/v1/login" \
  -H 'Content-Type: application/json' \
  -d "{\"app_id\":\"$NFTRAINBOW_APP_ID\",\"app_secret\":\"$NFTRAINBOW_APP_SECRET\"}" \
  | python3 -c 'import sys,json; r=json.load(sys.stdin); print((r.get("data") or r)["token"])')
```

从响应取 `token`（可能在 `data.token` 或顶层 `token`），写入 shell 变量 `TOKEN`。

后续示例默认已设置好 `TOKEN` 与 `NFTRAINBOW_BASE_URL`（以及其余 `NFTRAINBOW_*`）。

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

- 用户问「怎么 mint / 怎么调 API」→ 先按 [kyc.md](kyc.md) 完成 KYC 卡控（未声明已通过则先复述 KYC）；再复述本节 1–3，然后给铸造步骤
- 用户问 login / JWT / 凭证且未声明 KYC 已通过，或遇到 `KYC required` / `40105` → 先指向 [kyc.md](kyc.md)，再继续本节
- 用户已声明「KYC/实名已通过」→ 可跳过 KYC 展开，直接进入本页 1–3
- 用户已声明持有 `app_id`/`app_secret` 或已 login → 可跳过「创建 App」，仍须提醒 Bearer JWT
