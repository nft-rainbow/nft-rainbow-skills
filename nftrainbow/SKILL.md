---
name: nftrainbow
description: >
  NFTRainbow（NFT 彩虹桥）官方文档与 Open API 集成助手。在用户提及 NFTRainbow、
  Rainbow API、NFT 铸造、合约代付、POAP、Web3 Services，或在 NFTRainbow / Rainbow
  语境下提及 KYC、实名、setup、.env、40105、KYC required 时使用。
---

# NFTRainbow

同时服务：**文档导航**与 **Open API 集成**。API 标识符保持英文。

## 决策树

```
只说“想铸造 NFT”？       → 先执行下方「铸造入口」
文档 / 概念 / 链接？     → reference/docs.md
KYC / 实名 / setup？     → reference/setup.md
鉴权 / 凭证 / login / JWT？ → 先 setup.md，再 auth.md
要写集成代码？           → api.md + workflows.md + examples/
Web3 Services？          → web3-services.md
Activity 创建/发行（项目方）？ → 先 setup.md §1，再 activity.md
Activity 用户领取？          → activity.md
已知坑 / 硬性规则？      → pitfalls.md
Conflux 链基础概念？     → 可建议 [conflux-docs](https://github.com/conflux-fans/conflux-skills/blob/main/conflux-docs/SKILL.md)；Rainbow 代付步骤仍用本 skill
```

## API 要点

- Base：`https://api.nftrainbow.cn`
- 链：`conflux` / `conflux_test`
- 任务 `status`：0 pending / 1 success / 2 failed
- 铸造前确保代付/自动代付就绪

## 铸造入口

用户表达“想铸造 NFT”时，先说明两种路径，再只问一个分流问题：“你想最快试铸一枚，还是部署自己的 NFT 合约/系列？”

- 最快试铸 → Easy Mint：平台处理文件、metadata 和合约（[examples/easy-mint.md](examples/easy-mint.md)）
- 自有合约/系列 → Deploy → Sponsor → Mint：自选 ERC721/ERC1155，管理 metadata、转让规则和代付（[examples/deploy-sponsor-mint.md](examples/deploy-sponsor-mint.md)）

用户选择后按以下顺序执行：

1. **Setup**：按 [reference/setup.md](reference/setup.md) 执行；完成条件是约定位置存在 `.env`，且 `NFTRAINBOW_APP_ID` 与 `NFTRAINBOW_APP_SECRET` 均非空。
2. **鉴权**：按 [reference/auth.md](reference/auth.md) 执行；完成条件是成功取得 JWT。
3. **收集铸造信息**：两条路径都需要链、接收地址和 NFT 内容；自有合约还需要 ERC721/ERC1155、名称、symbol、转让规则和代付方式。完成条件是该路径的必需信息齐全；随后才提交链上任务。

## 禁止

- 臆造文档 URL，或编造 Activity / POAP / KYC 查询 Open API
- 禁止使用 Anyweb

## 文件索引

| 文件 | 用途 |
|------|------|
| [reference/docs.md](reference/docs.md) | 文档 / 概念；必要时再拉 GitBook `.md` 或 `?ask=` |
| [reference/setup.md](reference/setup.md) | 写集成的第一步；Activity/POAP **创建/发行** 前的 KYC |
| [reference/auth.md](reference/auth.md) | setup 之后：鉴权 / JWT |
| [reference/api.md](reference/api.md) | Open API 端点与字段概览 |
| [reference/workflows.md](reference/workflows.md) | 集成流程 |
| [examples/](examples/) | 可执行示例 |
| [reference/web3-services.md](reference/web3-services.md) | Web3 Services（仅购买/配置） |
| [reference/activity.md](reference/activity.md) | Activity / POAP；**用户领取**可直接读（领取方无需项目方 KYC） |
| [reference/pitfalls.md](reference/pitfalls.md) | 坑与硬性规则 |
| [openapi/swagger-2.0.json](openapi/swagger-2.0.json) | 通过局部查找获取字段细节，禁止整体读取（整读规则见 [reference/pitfalls.md](reference/pitfalls.md)） |
