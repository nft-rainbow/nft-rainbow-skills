# Easy Mint 示例

先设置好 `NFTRAINBOW_*` 环境变量（见 `env.md`）。

## 1. Login

响应形如 `{ "data": { "token": "...", "expire": "..." } }` 或顶层含 `token`；取 token：

```bash
TOKEN=$(curl -fsS -X POST "$NFTRAINBOW_BASE_URL/v1/login" \
  -H 'Content-Type: application/json' \
  -d "{\"app_id\":\"$NFTRAINBOW_APP_ID\",\"app_secret\":\"$NFTRAINBOW_APP_SECRET\"}" \
  | python3 -c 'import sys,json; r=json.load(sys.stdin); print((r.get("data") or r)["token"])')
```

## 2. Easy mint by URL

```bash
curl -fsS -X POST "$NFTRAINBOW_BASE_URL/v1/mints/easy/urls" \
  -H "Authorization: Bearer $TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{
    "chain": "conflux_test",
    "name": "Demo NFT",
    "description": "skill example",
    "file_url": "https://example.com/image.png",
    "mint_to_address": "cfxtest:YOUR_ADDRESS"
  }'
```

记下返回任务 `id`。

## 3. 轮询状态

```bash
curl -fsS -H "Authorization: Bearer $TOKEN" \
  "$NFTRAINBOW_BASE_URL/v1/mints/TASK_ID"
```

`status`：0 pending / 1 success / 2 failed。
