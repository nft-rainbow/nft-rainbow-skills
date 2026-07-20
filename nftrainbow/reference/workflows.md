# 工作流

开始前先完成鉴权前置（创建 App → `app_id`/`app_secret` → login → Bearer），见 [auth.md](auth.md)。
凭证环境变量见 `examples/env.md`。接口细节见 `reference/api.md`；字段以 pin swagger 为准。

## 1. Easy Mint（最快试铸）

适用：无自有合约、快速验证。

0. 完成鉴权前置（见 [auth.md](auth.md)）。
1. `POST /v1/login` 取 JWT。
2. 任选：
   - `POST /v1/mints/easy/files`（multipart：file + name/description/chain/mint_to_address）
   - `POST /v1/mints/easy/urls`（JSON metadata parts）
3. `GET /v1/mints/{id}` 轮询至 success/failed。
4. 失败可评估 `POST /v1/mints/{id}/reMint`（见 swagger 说明）。

文档：铸造流程 https://docs.nftrainbow.xyz/tutorials/mints/mints-zh

## 2. 自有合约：Deploy → Sponsor → Mint

0. 完成鉴权前置（见 [auth.md](auth.md)）。
1. Login。
2. `POST /v1/contracts` 部署 ERC721/ERC1155（`chain`/`name`/`symbol`/`type`）。
3. 等待合约 `status=1`；记录 `address`。
4. **代付：** `POST /v1/contracts/{address}/sponsor` 和/或自动代付配置 `.../config/auto-sponsor`。控制台说明：https://docs.nftrainbow.xyz/tutorials/guides/kong-zhi-tai-he-yue-dai-fu-she-zhi
5. 可选：`POST /v1/files/oss` → `POST /v1/metadata` 得到 metadata URI。
6. `POST /v1/mints`（CustomMint）或 batch；轮询任务状态。

## 3. 批量铸造要点

- API：`POST /v1/mints/customizable/batch` 等（见 api.md）。
- 控制台按序铸造有模板与数量上限说明：https://docs.nftrainbow.xyz/tutorials/guides/common-mint-mode

## 4. 转账 / 销毁

- Transfer：`POST /v1/transfers/customizable`（及 batch）；查 `GET /v1/transfers/{id}`。
- Burn：`POST /v1/burns`（及 batch）；查 `GET /v1/burns/{id}`。

## 5. 控制台 vs API

| 需求 | 控制台 | API |
|------|--------|-----|
| 非编程单次/批量铸 | 智能合约 → 铸造 NFT | Easy/Custom mint |
| 自动化集成 | — | `/v1/*` + JWT |
| 代付 | 控制台代付设置 | sponsor / auto-sponsor API |

## 6. 查状态与重试

- Mint 详情 / 列表：`GET /v1/mints`、`GET /v1/mints/{id}`
- `status`：0 pending / 1 success / 2 failed
- `reMint`：将失败任务重置为可再铸（仅在业务允许时）
