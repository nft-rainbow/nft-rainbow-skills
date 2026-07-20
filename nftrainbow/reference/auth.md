# 鉴权前置（Open API）

问 mint / 写 `/v1/*` 集成时，**必须先完成**本页；再进入 Easy Mint 或 Deploy → Sponsor → Mint。

## 1. 创建 App 并获取凭证

1. 注册并登录 [Rainbow 控制台](https://console.nftrainbow.cn)
2. 创建应用（App）
3. 取得 `app_id` 与 `app_secret`

环境变量写法见 [examples/env.md](../examples/env.md)。**禁止**把真实 secret 写入仓库。

## 2. Login 换 JWT

`POST https://api.nftrainbow.cn/v1/login`

Body：`{"app_id":"...","app_secret":"..."}`

从响应取 `token`（可能在 `data.token` 或顶层 `token`）。

```bash
curl -fsS -X POST "https://api.nftrainbow.cn/v1/login" \
  -H 'Content-Type: application/json' \
  -d '{"app_id":"YOUR_APP_ID","app_secret":"YOUR_APP_SECRET"}'
```

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

- 用户问「怎么 mint / 怎么调 API」→ 先复述本节 1–3，再给铸造步骤
- 用户已声明持有 `app_id`/`app_secret` 或已 login → 可跳过「创建 App」，仍须提醒 Bearer JWT
