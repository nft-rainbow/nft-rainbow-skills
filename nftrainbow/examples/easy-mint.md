# Easy Mint 示例

先完成 KYC（见 [../reference/kyc.md](../reference/kyc.md)），再完成鉴权前置并取得 `TOKEN`（含环境变量，见 [../reference/auth.md](../reference/auth.md)）。

## 1. Easy mint by URL

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

## 2. 轮询状态

```bash
curl -fsS -H "Authorization: Bearer $TOKEN" \
  "$NFTRAINBOW_BASE_URL/v1/mints/TASK_ID"
```

`status`：0 pending / 1 success / 2 failed。
