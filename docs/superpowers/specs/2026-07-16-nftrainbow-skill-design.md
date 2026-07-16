# NFTRainbow Skill 设计规格

**日期：** 2026-07-16  
**状态：** 已批准（实现计划见 `docs/superpowers/plans/2026-07-16-nftrainbow-skill.md`）  
**仓库根目录：** 即 skill 根目录（`nft-rainbow-skills/`）

## 1. 目标与边界

### 1.1 目标

在当前仓库根目录交付**完整可用的单体 skill**（非阶段性半成品），名称建议：`nftrainbow`。

同时服务两类同等重要的场景：

1. **文档导航** — 给出已验证的官方链接，说明产品能力与流程，禁止臆造 URL。
2. **API 集成** — 指导鉴权、调用、异步任务轮询与常见错误处理，帮助开发者写集成代码。

### 1.2 范围内（v1 成品）

| 产品线 | 范围 |
|--------|------|
| Rainbow 铸造工具 | 文档导航 + Open API 主链路（鉴权、文件、元数据、合约部署、代付、铸造/转账/销毁、NFT 查询） |
| Web3 Services | **仅**如何购买、如何配置；**不含** RPC/SCAN API 调用细节 |
| Rainbow Activity | **占位**：一句话能力说明 + 资料未齐、禁止编造实现细节 |

### 1.3 明确不做

- 不把 GitBook 全文或 `llms-full.txt` 塞进 skill
- 不在本 skill 内生成 typed SDK
- TBA、通用 Transaction 构建**不进入**精简 API 索引（需要时查 pin 的 swagger）
- 不维护 Anyweb；钱包相关指向晒啦钱包
- 不引入「先 MVP 再补」的临时结构；Activity 占位是**成品边界**（上游资料未齐），不是未完成交付

### 1.4 语言

- 正文与说明：**中文为主**
- API 路径、字段名、`operationId`、枚举值：保持**英文原文**

## 2. 架构选型

采用**单体 skill + `reference/` 渐进披露**（相对「多 skill 家族」或「全量文档 corpus 入库」）。

理由：文档与 API 使用场景重叠；仓库即 skill 顶层；Swagger 已有完整 `doc.json`，无需 strict-api-extraction 从零抽取。

## 3. 目录结构与文件职责

```
nft-rainbow-skills/
├── SKILL.md
├── reference/
│   ├── docs.md
│   ├── api.md
│   ├── workflows.md
│   ├── web3-services.md
│   ├── activity.md
│   └── pitfalls.md
├── openapi/
│   └── swagger-2.0.json
├── examples/
│   ├── env.md
│   ├── easy-mint.md
│   └── deploy-sponsor-mint.md
└── docs/
    ├── manual/                          # 既有资料/需求，不作为 agent 主路径
    └── superpowers/specs/               # 本设计规格等
```

### 3.1 读文件规则（写入 SKILL.md）

1. 先按 `SKILL.md` 决策树分流。
2. 文档问题 → `reference/docs.md`；必要时再拉 GitBook `.md` 或 `?ask=`。
3. API 集成 → `reference/api.md` + `reference/workflows.md` + `examples/`；字段级细节再**局部**查阅 pin swagger。
4. **禁止默认整读** `openapi/swagger-2.0.json`（避免上下文膨胀）。
5. 引用保持**一层深度**（`SKILL.md` → `reference/*.md`），不再嵌套更深路径作为主依赖。

## 4. SKILL.md 行为

### 4.1 Frontmatter

- `name`: `nftrainbow`
- `description`: 第三人称；同时写清 WHAT（文档 + Open API 集成）与 WHEN（触发词：NFTRainbow、Rainbow API、NFT 铸造、合约代付、元数据、`api.nftrainbow.cn`、`docs.nftrainbow.xyz`、Web3 Services 购买/配置、Activity 边界说明等）

### 4.2 意图分流

```
文档 / 概念 / 链接？     → reference/docs.md（必要时 GitBook）
要写集成代码？           → api.md + workflows.md + examples/
Web3 Services？          → reference/web3-services.md（仅购买/配置）
Activity / POAP？        → reference/activity.md（占位，不实现）
Conflux 代付 / 链基础？  → 本 skill 讲 Rainbow 侧步骤；链概念可指向 conflux-docs
```

### 4.3 文档获取（禁止臆造 URL）

1. 优先使用 `reference/docs.md` 中的已验证链接。
2. 页面 Markdown：`https://docs.nftrainbow.xyz/<path>.md`
3. 细粒度问答：在页面 URL 上使用 `?ask=&goal=`
4. 索引：`https://docs.nftrainbow.xyz/llms.txt`；路径不明时用 `sitemap.md`
5. 已知纠正：文档中的 Anyweb 已不可用 → 晒啦钱包

资料源头参考：`docs/manual/material-list.md`、GitBook 仓库 `https://github.com/nft-rainbow/rainbow-doc`。

### 4.4 API 行为

| 项 | 约定 |
|----|------|
| Base URL | `https://api.nftrainbow.cn` |
| 鉴权 | `POST /v1/login`（`app_id` + `app_secret`）→ `Authorization: Bearer <token>`；可用 refresh |
| 凭证 | 占位符 + 环境变量 `NFTRAINBOW_APP_ID` / `NFTRAINBOW_APP_SECRET`；禁止写入仓库或提交真实密钥 |
| 响应 | 业务数据关注 `data` 字段（与官方 swagger 说明一致） |
| 任务状态 | `status`：0 pending / 1 success / 2 failed；需轮询 |
| 链参数 | `conflux` / `conflux_test` |
| 铸造前置 | 合约须代付或自动代付就绪 |
| Swagger 使用 | 日常读 `reference/api.md`；字段细节 Grep/局部读 pin；禁止默认整读 |

### 4.5 推荐默认路径

- 快速试铸 → Easy Mint（`/v1/mints/easy/*`）
- 自有合约 → Login → Deploy → Sponsor →（Files / Metadata）→ Mint → 查任务状态

### 4.6 Pin swagger 更新策略

- **方式：** 手动下载覆盖 `openapi/swagger-2.0.json`
- **元数据：** `reference/api.md` 顶部写明上游 URL 与上次同步日期（可选 content-hash）
- **运行时规则：** 默认以本地 pin 为准；用户反馈接口与行为不符时，核对线上 `https://api.nftrainbow.cn/swagger/doc.json`，并提示更新 pin 与同步日期
- 本版**不要求** sync 脚本（后续若同步频繁可再加）

## 5. reference / examples 内容要点

### 5.1 `reference/docs.md`

按产品线列出已验证文档链接：首页、铸造工具简介、交互流程、铸造流程、账户方案（标注 Anyweb 废弃）、元数据管理、代付设置、Web3 Services 产品页、社区指南；并说明 `llms.txt` / `sitemap.md` 用法。

### 5.2 `reference/api.md`

- 顶部：上游 URL + 上次同步日期
- 正文仅主链路：Login、Files、Metadata、Contract（含 sponsor）、Mints、Transfers、Burns、NFT 查询
- 每条：method、path、`operationId`、一句话用途
- 文末：TBA / Transaction「见 pin swagger，非本 skill 重点」

### 5.3 `reference/workflows.md`

1. Easy Mint（file / url）
2. 部署合约 → 代付 / 自动代付 → 自定义铸造（含批量要点）
3. 查询 mint / tx 状态与失败重试（如 `reMint`）
4. 控制台铸造工具路径与 API 路径对照

### 5.4 `reference/web3-services.md`

产品一句话、如何购买、如何配置到可用；不写 RPC/SCAN 调用细节；链到官方产品文档。

### 5.5 `reference/activity.md`

能力一句话 +「资料未齐，禁止编造领取/发行 API 或晒啦对接细节」+ 待补资料提示。

### 5.6 `reference/pitfalls.md`

代付前置、JWT 过期、异步 status、链枚举、Anyweb→晒啦、勿整读 swagger、pin 与线上不一致时的处理。

### 5.7 `examples/`

- `env.md`：环境变量约定
- `easy-mint.md`：login → easy mint → 查状态
- `deploy-sponsor-mint.md`：deploy → sponsor → mint 最小闭环

示例中凭证一律占位符或环境变量名，不含真实密钥。

## 6. 验证与验收

实现完成后，用固定场景验收 skill 行为：

1. 「怎么用 Rainbow 两分钟铸 NFT」→ Easy Mint + 正确文档链接
2. 「部署合约再铸造」→ 含代付步骤，不跳步
3. 「Web3 Services 怎么买/配」→ 只谈购买与配置
4. 「做个 Activity/POAP」→ 走占位，不编造 API
5. 抽一个主链路 endpoint 写请求体 → 先 `api.md` 再局部查 pin，字段与 swagger 一致

另检：`SKILL.md` 少于 500 行；description 含 WHAT + WHEN；术语一致；引用一层深。

## 7. 实现顺序（供后续 writing-plans）

1. 拉取并 pin `openapi/swagger-2.0.json`
2. 编写 `reference/api.md`（含同步元数据）与其余 `reference/*`
3. 编写 `examples/*`
4. 编写 `SKILL.md`
5. 按第 6 节场景做验收自测

## 8. 决策记录

| 决策点 | 结论 |
|--------|------|
| 文档 vs API | 同等重要（单体 skill） |
| 产品范围 | 铸造全链路 + Web3 购买/配置 + Activity 占位 |
| 目录 | `reference/` 文件夹，非根目录 `reference-*.md` |
| 语言 | 中文为主 |
| 布局 | 仓库根 = skill 根 |
| Swagger | pin 全量 + 精简 api.md 主入口 |
| Pin 更新 | 手动覆盖 + api.md 顶部同步信息 + SKILL 过期核对规则 |
| 凭证 | 占位符 + `NFTRAINBOW_APP_ID` / `NFTRAINBOW_APP_SECRET` |
| API 精简索引 | 仅铸造主链路（不含 TBA / Transaction 构建） |
