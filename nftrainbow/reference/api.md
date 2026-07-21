# NFTRainbow Open API 精简索引

- **上游 URL:** `https://api.nftrainbow.cn/swagger/doc.json`
- **上次同步日期:** 2026-07-16
- **本地 pin:** `openapi/swagger-2.0.json`
- **pin sha256:** `a4aba78f81100f38fe74cc4b6c8324285a27a58428a02d4226b5125b46044cd0`
- **Base URL:** `https://api.nftrainbow.cn`

## 使用规则

1. 日常先查本文件；写请求体/字段时再对 pin swagger **局部** Grep/`operationId` 定位，禁止默认整读。
2. 鉴权前置（强制）：在 [Rainbow 控制台](https://console.nftrainbow.cn) 创建 App 获取 `app_id`/`app_secret`，并按 [auth.md](auth.md) 完成 login 与 Bearer。
3. 凭证环境变量示例见 `examples/env.md`。
4. 链枚举：`conflux` / `conflux_test`。
5. 异步任务 `status`：`0` pending / `1` success / `2` failed。
6. 业务响应关注 `data` 字段。
7. 精简索引以 `/v1/*` App Open API 为主；若含 `/dashboard/*`，见行内标注。

## Login

| Method | Path | operationId | 用途 |
|--------|------|-------------|------|
| `POST` | `/dashboard/login` | `UserLogin` | User login（Dashboard/用户态；集成优先 `/v1/*`） |
| `GET` | `/dashboard/refresh_token` | `RefreshUserAuth` | Refresh JWT（Dashboard/用户态；集成优先 `/v1/*`） |
| `POST` | `/v1/login` | `LoginApp` | App login |
| `GET` | `/v1/refresh_token` | `RefreshAppAuth` | Refresh JWT |

## Files

| Method | Path | operationId | 用途 |
|--------|------|-------------|------|
| `GET` | `/v1/files` | `ListFiles` | Obtain file list |
| `POST` | `/v1/files/folder/oss` | `UploadFolderToOSS` | Upload folder to oss |
| `POST` | `/v1/files/oss` | `UploadFileToOss` | Upload file to OSS |

## Metadata

| Method | Path | operationId | 用途 |
|--------|------|-------------|------|
| `GET` | `/v1/metadata` | `ListMetadatas` | Obtain metadata list |
| `POST` | `/v1/metadata` | `CreateMetadata` | Create NFT metadata |
| `GET` | `/v1/metadata/{metadata_id}` | `GetMetadatInfo` | Query metadata |

## Contract

| Method | Path | operationId | 用途 |
|--------|------|-------------|------|
| `GET` | `/v1/contracts` | `ListContracts` | Obtain contract list |
| `POST` | `/v1/contracts` | `DeployContract` | Deploy contract |
| `GET` | `/v1/contracts/detail/{id}` | `GetContractInfo` | Contract detail |
| `GET` | `/v1/contracts/{address}/admin` | `GetContractAdmin` | Get administrator of contract, only work on conflux chain |
| `PUT` | `/v1/contracts/{address}/admin` | `UpdateContractAdmin` | Update administrator of contract |
| `GET` | `/v1/contracts/{address}/config/auto-sponsor` | `GetContractAutoSponsor` | Get contract auto sponsor config |
| `POST` | `/v1/contracts/{address}/config/auto-sponsor` | `SetContractAutoSponsor` | Set contract auto sponsor config |
| `POST` | `/v1/contracts/{address}/config/transaferable` | `SetContractTransaferable` | Set Contract Transaferable Config |
| `GET` | `/v1/contracts/{address}/profile` | `GetContractProfile` | Get contract runtime profile |
| `GET` | `/v1/contracts/{address}/sponsor` | `GetContractSponsorInfo` | Query sponsor |
| `POST` | `/v1/contracts/{address}/sponsor` | `SetContractSponsor` | Set sponsor |
| `GET` | `/v1/contracts/{address}/sponsor/whitelist` | `GetContractSponsoredWhitelist` | Get contract sponsored whitelist |
| `POST` | `/v1/contracts/{address}/sponsor/whitelist` | `AddContractSponsorWhitelist` | Add contract sponsored whitelist |
| `DELETE` | `/v1/contracts/{address}/sponsor/whitelist` | `RemoveContractSponsorWhitelist` | Remove contract sponsored whitelist |

## Mints

| Method | Path | operationId | 用途 |
|--------|------|-------------|------|
| `POST` | `/dashboard/apps/{id}/nft/batch/by-meta-parts` | `AppBatchMintNFT` | Batch Mint NFT with metadata parts（Dashboard/用户态；集成优先 `/v1/*`） |
| `POST` | `/dashboard/apps/{id}/nft/batch/by-meta-uri` | `AppBatchMintByMetaUri` | Batch Mint NFT with metadata uri（Dashboard/用户态；集成优先 `/v1/*`） |
| `GET` | `/v1/mints` | `ListMints` | Obtain NFT list |
| `POST` | `/v1/mints` | `CustomMint` | Mint NFT |
| `POST` | `/v1/mints/customizable/batch` | `BatchCustomMint` | Batch Mint NFTs |
| `POST` | `/v1/mints/easy/files` | `EasyMintByFile` | Mint NFT with file |
| `POST` | `/v1/mints/easy/urls` | `EasyMintByMetadata` | Mint NFT with metadata |
| `GET` | `/v1/mints/{id}` | `GetMintDetail` | Mint NFT detail |
| `POST` | `/v1/mints/{id}/reMint` | `ReMintNFT` | Reset mint task status to init so that it can be minted again |

## Transfers

| Method | Path | operationId | 用途 |
|--------|------|-------------|------|
| `GET` | `/v1/transfers` | `ListTransfer` | Obtain the transferred NFTs list |
| `POST` | `/v1/transfers/customizable` | `TransferNft` | Transfer NFT |
| `POST` | `/v1/transfers/customizable/batch` | `BatchTransferNft` | Batch Transfer NFTs |
| `GET` | `/v1/transfers/{id}` | `GetTransferDetail` | Transfer NFT detail |

## Burns

| Method | Path | operationId | 用途 |
|--------|------|-------------|------|
| `GET` | `/v1/burns` | `GetBurnList` | Obtain the burned NFTs list |
| `POST` | `/v1/burns` | `BurnNft` | Burn NFT |
| `POST` | `/v1/burns/customizable/batch` | `BurnBatch` | Batch burn NFT |
| `GET` | `/v1/burns/{id}` | `GetBurnDetail` | Burn NFT detail |

## NFTs

| Method | Path | operationId | 用途 |
|--------|------|-------------|------|
| `GET` | `/v1/nft/count/{address}/{token_id}/{holder}` | `NFTHoldCount` | Get NFT hold count by address |
| `GET` | `/v1/nft/{address}/{token_id}` | `NftInfo` | Get NFT info, mainly owner and metadata |
| `PUT` | `/v1/nft/{address}/{token_id}/tokenUri` | `UpdateNftTokenUri` | Update NFT token uri |

## Accounts

铸造到手机号/托管账户时可用。

| Method | Path | operationId | 用途 |
|--------|------|-------------|------|
| `GET` | `/v1/accounts` | `QueryAccounts` | Query web3 account |
| `POST` | `/v1/accounts` | `InsertAccount` | Insert web3 account |

## 非本 skill 重点

以下 tag **不在**本索引展开，需要时查 `openapi/swagger-2.0.json`：

- **TBA**（ERC-6551）
- **Transaction**（通用 send / build approve / transfer 等）

