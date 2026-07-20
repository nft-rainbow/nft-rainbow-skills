# Deploy → Sponsor → Custom Mint

先完成鉴权前置（创建 App → login → Bearer），见 [../reference/auth.md](../reference/auth.md)。
环境变量同 `env.md`。先获取 `TOKEN`（同 `easy-mint.md`）。

## 1. Deploy

```bash
curl -fsS -X POST "$NFTRAINBOW_BASE_URL/v1/contracts" \
  -H "Authorization: Bearer $TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{
    "chain": "conflux_test",
    "name": "DemoCollection",
    "symbol": "DEMO",
    "type": "erc721",
    "auto_sponsor": true
  }'
```

轮询 `GET /v1/contracts/detail/{id}` 至 `status=1`，取 `address`。

## 2. Sponsor（若未自动代付或需补设）

```bash
curl -fsS -X POST \
  "$NFTRAINBOW_BASE_URL/v1/contracts/CONTRACT_ADDRESS/sponsor?chain=conflux_test&auto_sponsor=true" \
  -H "Authorization: Bearer $TOKEN"
```

字段以 pin swagger `SetContractSponsor` 为准。

## 3. Custom mint

```bash
curl -fsS -X POST "$NFTRAINBOW_BASE_URL/v1/mints" \
  -H "Authorization: Bearer $TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{
    "chain": "conflux_test",
    "contract_address": "CONTRACT_ADDRESS",
    "mint_to_address": "cfxtest:YOUR_ADDRESS",
    "metadata_uri": "https://example.com/metadata.json"
  }'
```

## 4. 查 mint

`GET /v1/mints/{id}` 直到 success/failed。
