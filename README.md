# nft-rainbow-skills

> NFTRainbow（NFT 彩虹桥）文档导航与 Open API 集成 Agent Skill

在 Cursor / Claude Code / Codex 等环境中，用官方文档与 pin 过的 Swagger，协助完成铸造、合约部署、代付与相关集成，避免臆造接口或过时链接。

[Features](#features) · [Installation](#installation) · [Usage](#usage) · [Examples](#examples)

## Features

- **文档导航** — 指向 [docs.nftrainbow.xyz](https://docs.nftrainbow.xyz/) 已验证页面（支持 `.md` / `?ask=`）
- **Open API 集成** — 鉴权、部署合约、代付、元数据、铸造 / 转账 / 销毁主链路
- **渐进披露** — 日常用精简 `reference/api.md`；字段细节再局部查 pin 的 swagger
- **示例脚本** — curl + 环境变量的 Easy Mint 与 Deploy → Sponsor → Mint 闭环
- **Rainbow Activity / POAP** — 控制台发行、领取页与晒啦领取流程（非 Open API）
- **明确边界** — Web3 Services 仅购买/配置；勿编造 Activity API 或 RPC/SCAN 调用细节

| 产品线 | 本 skill 覆盖 |
|--------|----------------|
| 铸造工具 | 文档 + Open API 主链路 |
| Web3 Services | 购买与配置 only |
| Activity / POAP | 控制台创建活动 + 用户领取流程 |

## Installation

```sh
npx skills add https://github.com/nft-rainbow/nft-rainbow-skills --skill nftrainbow
```

全局安装：

```sh
npx skills add https://github.com/nft-rainbow/nft-rainbow-skills --skill nftrainbow -g
```

> [!NOTE]
> Skills CLI 会对仓库做浅克隆（`--depth=1`）。网络较慢时，可先 clone 到本地，再执行：
> `npx skills add ./nft-rainbow-skills -g`

安装后，对话中出现 NFTRainbow、Rainbow API、NFT 铸造、合约代付、`api.nftrainbow.cn`、`docs.nftrainbow.xyz` 等关键词即可触发。

## Usage

Skill 在 `nftrainbow/` 目录下，**文件夹名与 frontmatter `name: nftrainbow` 一致**：

```text
.
├── README.md
└── nftrainbow/
    ├── SKILL.md              # 触发条件、决策树、硬性规则
    ├── reference/            # 文档索引、鉴权前置、API、工作流、产品边界
    ├── examples/             # env + Easy Mint + 部署代付铸造
    └── openapi/
        └── swagger-2.0.json  # pin 的官方 Swagger 2.0（~188KB）
```

集成时建议：

1. 先在 [Rainbow 控制台](https://console.nftrainbow.cn) 创建 App 并取得 `app_id` / `app_secret`
2. 先完成鉴权前置（见 [`nftrainbow/reference/auth.md`](nftrainbow/reference/auth.md)），再调 `/v1/*`
3. 设置 `NFTRAINBOW_APP_ID` / `NFTRAINBOW_APP_SECRET`（见 [`nftrainbow/examples/env.md`](nftrainbow/examples/env.md)）
4. 链参数使用 `conflux` / `conflux_test`
5. 铸造前确保合约代付或自动代付就绪
6. 轮询任务 `status`：`0` pending / `1` success / `2` failed

> [!IMPORTANT]
> 不要把真实 `app_secret` 写入仓库。字段级请求体以 [`nftrainbow/openapi/swagger-2.0.json`](nftrainbow/openapi/swagger-2.0.json) 为准，禁止默认整读该文件。

上游：

- 文档：https://docs.nftrainbow.xyz/
- API：https://api.nftrainbow.cn/（Swagger：`/swagger/doc.json`）

## Examples

在 Agent 对话中可以直接问：

```text
用 Rainbow Easy Mint 在 conflux_test 铸一枚 NFT
```

```text
部署 ERC721，设置代付，再铸造到指定地址
```

```text
Web3 Services 怎么购买套餐，以及在控制台哪里拿 key / URL？
```

```text
怎么用 Rainbow 发一个 POAP 活动，用户怎么领取？
```

最后一例按 [`nftrainbow/reference/activity.md`](nftrainbow/reference/activity.md) 说明控制台与领取页流程，不会编造 Activity Open API。

手动对照示例：

- [`nftrainbow/examples/easy-mint.md`](nftrainbow/examples/easy-mint.md)
- [`nftrainbow/examples/deploy-sponsor-mint.md`](nftrainbow/examples/deploy-sponsor-mint.md)

## Updating the pinned Swagger

```sh
curl -fsSL -o nftrainbow/openapi/swagger-2.0.json \
  https://api.nftrainbow.cn/swagger/doc.json
```

然后更新 `nftrainbow/reference/api.md` 顶部的同步日期与 sha256，并按需刷新精简索引表。
