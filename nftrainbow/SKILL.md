---
name: nftrainbow
description: >
  NFTRainbow（NFT 彩虹桥）官方文档与 Open API 集成助手。覆盖铸造工具文档导航、
  合约部署/代付/元数据/铸造/转账/销毁、Web3 Services 购买与配置、以及 Rainbow Activity/POAP
  控制台发行与领取流程。在用户提及 NFTRainbow、Rainbow API、NFT 铸造、合约代付、元数据、
  api.nftrainbow.cn、docs.nftrainbow.xyz、Web3 Services、Activity 或 POAP 时使用。
---

# NFTRainbow

同时服务：**文档导航**与 **Open API 集成**。中文说明；API 标识符保持英文。

## 读文件规则

1. 先用下方决策树分流。
2. 文档 → [reference/docs.md](reference/docs.md)；必要时拉 GitBook `.md` 或 `?ask=`。
3. 写集成 → **先**读 [reference/auth.md](reference/auth.md)，再看 [reference/api.md](reference/api.md) + [reference/workflows.md](reference/workflows.md) + [examples/](examples/)。
4. 字段细节 → 对 [openapi/swagger-2.0.json](openapi/swagger-2.0.json) **局部**查找；**禁止默认整读**。
5. Web3 Services → [reference/web3-services.md](reference/web3-services.md)（仅购买/配置）。
6. Activity / POAP → [reference/activity.md](reference/activity.md)（控制台流程，非 Open API）。
7. 坑与硬性规则 → [reference/pitfalls.md](reference/pitfalls.md)。

## 决策树

```
文档 / 概念 / 链接？     → reference/docs.md
鉴权 / 凭证 / login / JWT？ → reference/auth.md
要写集成代码？           → 先 auth.md，再 api.md + workflows.md + examples/
Web3 Services？          → web3-services.md（禁止展开 RPC/SCAN 调用）
Activity / POAP？        → activity.md（控制台+领取页；禁止臆造 Activity API）
Conflux 链基础概念？     → 可建议 conflux-docs；Rainbow 代付步骤仍用本 skill
```

## API 要点

- Base：`https://api.nftrainbow.cn`
- 鉴权前置（强制）：创建 App → `app_id`/`app_secret` → `POST /v1/login` → `Authorization: Bearer <token>`（详见 [reference/auth.md](reference/auth.md)）
- 链：`conflux` / `conflux_test`
- 任务 `status`：0 pending / 1 success / 2 failed
- 铸造前确保代付/自动代付就绪
- Pin 默认可信；接口不符时核对线上 swagger 并提示更新 pin 与 `api.md` 同步信息

## 推荐路径

- 最快试铸 → Easy Mint（[examples/easy-mint.md](examples/easy-mint.md)）
- 自有合约 → Deploy → Sponsor → Mint（[examples/deploy-sponsor-mint.md](examples/deploy-sponsor-mint.md)）

## 禁止

- 臆造文档 URL，或编造 Activity / POAP Open API
- 推荐 Anyweb（用晒啦）
- 把秘密写入仓库
- 在 Web3 Services 场景输出 RPC/SCAN 调用教程
- 默认整读 swagger
