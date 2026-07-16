# NFTRainbow Skill Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 在仓库根目录交付完整可用的单体 skill `nftrainbow`（文档导航 + Open API 集成），覆盖铸造主链路、Web3 Services 购买/配置、Activity 占位。

**Architecture:** 仓库根 = skill 根；`SKILL.md` 做触发与决策树；`reference/` 渐进披露；`openapi/swagger-2.0.json` pin 全量官方 Swagger 2.0；日常 API 只读精简 `reference/api.md`，禁止默认整读 swagger。

**Tech Stack:** Markdown Agent Skill；官方文档 GitBook（`docs.nftrainbow.xyz`）；Open API 上游 `https://api.nftrainbow.cn/swagger/doc.json`；验证用 shell/`python3`/`jq`（无应用运行时依赖）。

**Spec:** `docs/superpowers/specs/2026-07-16-nftrainbow-skill-design.md`

## Global Constraints

- 语言：中文为主；路径、字段名、`operationId`、枚举保持英文原文
- 凭证：仅占位符 + `NFTRAINBOW_APP_ID` / `NFTRAINBOW_APP_SECRET`；禁止真实密钥入仓
- Web3 Services：只写购买与配置，不写 RPC/SCAN API 调用细节
- Activity：占位，禁止编造实现细节
- TBA / Transaction：不进精简 API 索引（见 pin swagger）
- Anyweb 已废弃 → 晒啦钱包
- `SKILL.md` 少于 500 行；引用一层深（`SKILL.md` → `reference/*.md` | `examples/*.md`）
- 不提交无关文件；不 push；每个 Task 结束做一次 commit

## File Structure

| 路径 | 职责 |
|------|------|
| `openapi/swagger-2.0.json` | pin 全量官方 swagger |
| `reference/api.md` | 精简 API 索引 + 同步元数据 |
| `reference/docs.md` | 已验证文档 URL 索引 |
| `reference/workflows.md` | 端到端铸造/代付工作流 |
| `reference/web3-services.md` | 购买与配置 |
| `reference/activity.md` | Activity 占位 |
| `reference/pitfalls.md` | 已知坑与硬性规则 |
| `examples/env.md` | 环境变量约定 |
| `examples/easy-mint.md` | Easy Mint 示例 |
| `examples/deploy-sponsor-mint.md` | 部署+代付+铸造示例 |
| `SKILL.md` | 触发、决策树、行为规则 |

---

### Task 1: Pin 官方 Swagger

**Files:**
- Create: `openapi/swagger-2.0.json`

**Interfaces:**
- Consumes: `https://api.nftrainbow.cn/swagger/doc.json`
- Produces: 本地 pin 文件，供 Task 2 生成/核对 `reference/api.md`

- [ ] **Step 1: 创建目录并下载**

```bash
mkdir -p openapi
curl -fsSL -o openapi/swagger-2.0.json \
  "https://api.nftrainbow.cn/swagger/doc.json"
```

- [ ] **Step 2: 校验 JSON 与规模**

```bash
python3 - <<'PY'
import json, hashlib, pathlib
p = pathlib.Path("openapi/swagger-2.0.json")
raw = p.read_bytes()
d = json.loads(raw)
assert d.get("swagger") == "2.0", d.get("swagger")
assert d.get("host") == "api.nftrainbow.cn"
assert "paths" in d and len(d["paths"]) >= 40
print("ok paths=", len(d["paths"]), "bytes=", len(raw))
print("sha256=", hashlib.sha256(raw).hexdigest())
PY
```

Expected: 打印 `ok paths=`（约 45）与 `sha256=`；无 AssertionError。

- [ ] **Step 3: Commit**

```bash
git add openapi/swagger-2.0.json
git commit -m "$(cat <<'EOF'
chore: pin NFTRainbow swagger 2.0 spec

Vendor the official Open API document for offline, reproducible API lookup.
EOF
)"
```

---

### Task 2: 编写 `reference/api.md`

**Files:**
- Create: `reference/api.md`

**Interfaces:**
- Consumes: `openapi/swagger-2.0.json`
- Produces: 主链路精简索引（Login / Files / Metadata / Contract / Mints / Transfers / Burns / NFTs；Accounts 作为铸造收款辅助一并收录）；顶部含上游 URL + 同步日期 + sha256

- [ ] **Step 1: 用脚本从 pin 生成草稿到 stdout，确认无 TBA/Transaction**

```bash
python3 - <<'PY'
import json, hashlib, pathlib
from datetime import date
raw = pathlib.Path("openapi/swagger-2.0.json").read_bytes()
d = json.loads(raw)
keep = {"Login","Files","Metadata","Contract","Mints","Transfers","Burns","NFTs","Accounts"}
exclude_tags = {"TBA","Transaction"}
rows = []
for path, methods in sorted(d["paths"].items()):
    for method, op in methods.items():
        if not isinstance(op, dict):
            continue
        tags = op.get("tags") or []
        if any(t in exclude_tags for t in tags):
            continue
        if not any(t in keep for t in tags):
            continue
        rows.append((tags[0], method.upper(), path, op.get("operationId",""), op.get("summary","")))
assert not any(r[0] in exclude_tags for r in rows)
print("rows", len(rows))
for tag in ["Login","Files","Metadata","Contract","Mints","Transfers","Burns","NFTs","Accounts"]:
    print("\n###", tag)
    for t,m,p,oid,s in rows:
        if t == tag:
            print(f"| `{m}` | `{p}` | `{oid}` | {s} |")
print("\nMETA", date.today().isoformat(), hashlib.sha256(raw).hexdigest())
PY
```

Expected: `rows` 约 46；无 TBA/Transaction 段。

- [ ] **Step 2: 写入 `reference/api.md`（内容必须与 pin 一致；日期用执行当日）**

创建 `reference/api.md`，结构如下（表格行用 Step 1 输出替换，勿手抄错 path）：

```markdown
# NFTRainbow Open API 精简索引

- **上游 URL:** `https://api.nftrainbow.cn/swagger/doc.json`
- **上次同步日期:** YYYY-MM-DD
- **本地 pin:** `openapi/swagger-2.0.json`
- **pin sha256:** `<Task1 打印的完整 hash>`
- **Base URL:** `https://api.nftrainbow.cn`

## 使用规则

1. 日常先查本文件；写请求体/字段时再对 pin swagger **局部** Grep/`operationId` 定位，禁止默认整读。
2. 鉴权：`POST /v1/login`，body `{"app_id","app_secret"}`，之后 `Authorization: Bearer <token>`。
3. 凭证环境变量：`NFTRAINBOW_APP_ID`、`NFTRAINBOW_APP_SECRET`。
4. 链枚举：`conflux` / `conflux_test`。
5. 异步任务 `status`：`0` pending / `1` success / `2` failed。
6. 业务响应关注 `data` 字段。

## Login

| Method | Path | operationId | 用途 |
|--------|------|-------------|------|
| ... | ... | ... | ... |

## Files
...

## Metadata
...

## Contract
...

## Mints
...

## Transfers
...

## Burns
...

## NFTs
...

## Accounts

铸造到手机号/托管账户时可用。

| Method | Path | operationId | 用途 |
|--------|------|-------------|------|
| ... | ... | ... | ... |

## 非本 skill 重点

以下 tag **不在**本索引展开，需要时查 `openapi/swagger-2.0.json`：

- **TBA**（ERC-6551）
- **Transaction**（通用 send / build approve / transfer 等）
- **Dashboard 用户登录**（`/dashboard/login`）一般集成用 App Login（`/v1/login`）；表格若含 dashboard 行，标注「控制台用户态，非 App Open API 主路径」
```

注意：精简索引以 `/v1/*` App Open API 为主。若 Step 1 包含 `/dashboard/*` mint/login，保留但在 Login/Mints 节用一句话标明「Dashboard 路径多为控制台/用户 JWT，集成优先 `/v1/*`」。

- [ ] **Step 3: 结构验收**

```bash
test -f reference/api.md
grep -q '上游 URL' reference/api.md
grep -q '上次同步日期' reference/api.md
grep -q 'openapi/swagger-2.0.json' reference/api.md
grep -q 'NFTRAINBOW_APP_ID' reference/api.md
grep -q '非本 skill 重点' reference/api.md
! grep -E '^\| `.*(TBA|BuildNft)' reference/api.md
wc -l reference/api.md
```

Expected: 全部 `grep -q` 成功；`wc -l` 合理（约 80–200）；无 TBA 表行误入主表（「非本 skill 重点」段落可提到 TBA 文字）。

- [ ] **Step 4: Commit**

```bash
git add reference/api.md
git commit -m "$(cat <<'EOF'
docs(skill): add slim Open API index with pin metadata

Document main-path endpoints and sync provenance for agent-safe API lookup.
EOF
)"
```

---

### Task 3: 编写其余 `reference/` 文档

**Files:**
- Create: `reference/docs.md`
- Create: `reference/workflows.md`
- Create: `reference/web3-services.md`
- Create: `reference/activity.md`
- Create: `reference/pitfalls.md`

**Interfaces:**
- Consumes: spec §5、`docs/manual/material-list.md`、官方文档页
- Produces: SKILL.md 决策树目标文件

- [ ] **Step 1: 写入 `reference/docs.md`**

```markdown
# NFTRainbow 文档索引

禁止臆造 URL。优先本表；需要正文时给页面加 `.md` 后缀。细粒度问答可用 `?ask=<问题>&goal=<目标>`。

## 入口与索引

| 说明 | URL |
|------|-----|
| 文档首页 | https://docs.nftrainbow.xyz/ |
| llms.txt | https://docs.nftrainbow.xyz/llms.txt |
| sitemap | https://docs.nftrainbow.xyz/sitemap.md |
| 文档源码仓库 | https://github.com/nft-rainbow/rainbow-doc |

## 铸造工具

| 主题 | URL |
|------|-----|
| Rainbow 铸造工具简介 | https://docs.nftrainbow.xyz/tutorials/guides/common-mint-mode |
| 交互流程图 | https://docs.nftrainbow.xyz/tutorials/interactive |
| 铸造流程 | https://docs.nftrainbow.xyz/tutorials/mints/mints-zh |
| 账户方案 | https://docs.nftrainbow.xyz/tutorials/account-solution |
| 元数据管理 | https://docs.nftrainbow.xyz/tutorials/guides/metadata-manage |
| 控制台合约代付设置 | https://docs.nftrainbow.xyz/tutorials/guides/kong-zhi-tai-he-yue-dai-fu-she-zhi |

**纠正：** 账户方案文档中的 Anyweb 已不可用；用户侧钱包请使用**晒啦**。

## 其他产品

| 主题 | URL |
|------|-----|
| Web3 Services | https://docs.nftrainbow.xyz/products/web3-services |
| Open API 相关（Authentication / Error codes 等） | 以 `llms.txt` / `sitemap.md` 为准，勿猜测路径 |

## 社区

| 主题 | URL |
|------|-----|
| 铸造工具 / 社区指南入口 | https://docs.nftrainbow.xyz/tutorials/guides/common-mint-mode |
```

- [ ] **Step 2: 写入 `reference/web3-services.md`**

```markdown
# Web3 Services：购买与配置

本 skill **只**覆盖购买与配置。不提供 RPC/SCAN 接口调用、参数、或迁移指南。

## 是什么

NFTRainbow 提供区块链 RPC 与数据索引（Scan 兼容）等基础设施，覆盖 Conflux Core / eSpace，主网与测试网。产品说明：https://docs.nftrainbow.xyz/products/web3-services

## 如何购买

1. 打开 Rainbow Console。
2. 进入 `Web3服务 -> 我的服务`。
3. 使用购买入口选择套餐或加油包。
4. 套餐可切换；取消自动续订后到期回免费版；切换立即生效且不退费（以官网 FAQ 为准）。

文档：https://docs.nftrainbow.xyz/products/web3-services.md（章节「如何购买服务？」）

## 如何配置（拿到可用访问）

1. 在 Console 的 **Web3服务** 页面查看或创建应用。
2. 每个应用分配访问 key；在应用详情查看 key 与完整服务 URL。
3. 将 key / URL 配置到你的项目环境（勿提交真实 key；可用本地环境变量，命名由项目自定）。

文档：同页 FAQ「如何获取服务访问 url 及 key？」

## Agent 禁区

- 不要展开 RPC method、Scan API path、示例 curl 调链上数据。
- 用户若追问接口细节：指向官方 Web3 Services 页中的 Core/eSpace RPC 与 Scan 文档链接，并说明超出本 skill 范围。
```

- [ ] **Step 3: 写入 `reference/activity.md`**

```markdown
# Rainbow Activity（占位）

## 能力（一句话）

项目方可通过网页端发行活动（POAP 也属于一种 Activity），用户通过钱包（晒啦）领取 NFT。

## 硬性规则

- **资料未齐**：本 skill **禁止**编造 Activity 发行/领取 API、晒啦对接步骤、或控制台操作细节。
- 用户要求「实现 Activity / POAP 对接」时：说明当前仅占位，请补充官方文档或内部资料后再扩展本文件。
- 钱包：使用晒啦；不要推荐 Anyweb。

## 待补资料

- Activity 官方教程完整链接与步骤
- 是否有 Open API、鉴权方式、状态查询
- 与铸造工具 Open API 的边界
```

- [ ] **Step 4: 写入 `reference/pitfalls.md`**

```markdown
# 已知坑与硬性规则

1. **代付前置：** 合约需设置代付或开启自动代付后才能稳定铸造；跳过代付是常见失败原因。
2. **JWT：** `app_id`/`app_secret` 换 token；过期用 refresh；勿把 secret 写入仓库。
3. **异步任务：** mint/deploy/transfer/burn 返回任务；轮询 `status`（0/1/2），不要假设同步上链完成。
4. **链枚举：** 只用 `conflux` / `conflux_test`（以 pin swagger 为准）。
5. **钱包：** Anyweb 不可用 → 晒啦。
6. **Swagger：** 禁止默认整读 `openapi/swagger-2.0.json`；先 `reference/api.md`。
7. **Pin 过期：** 默认信本地 pin；用户反馈接口不符时核对线上 `doc.json`，提示更新 pin 与 `api.md` 同步日期/sha256。
8. **Activity：** 见 `activity.md`，勿臆造。
9. **Web3 Services：** 本 skill 只谈购买/配置，不谈 RPC/SCAN 调用。
```

- [ ] **Step 5: 写入 `reference/workflows.md`**

```markdown
# 工作流

凭证见 `examples/env.md`。接口细节见 `reference/api.md`；字段以 pin swagger 为准。

## 1. Easy Mint（最快试铸）

适用：无自有合约、快速验证。

1. `POST /v1/login` 取 JWT。
2. 任选：
   - `POST /v1/mints/easy/files`（multipart：file + name/description/chain/mint_to_address）
   - `POST /v1/mints/easy/urls`（JSON metadata parts）
3. `GET /v1/mints/{id}` 轮询至 success/failed。
4. 失败可评估 `POST /v1/mints/{id}/reMint`（见 swagger 说明）。

文档：铸造流程 https://docs.nftrainbow.xyz/tutorials/mints/mints-zh

## 2. 自有合约：Deploy → Sponsor → Mint

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
```

- [ ] **Step 6: 验收**

```bash
for f in docs.md workflows.md web3-services.md activity.md pitfalls.md; do
  test -f "reference/$f" || exit 1
done
grep -q 'Anyweb' reference/docs.md
grep -q '晒啦' reference/docs.md
grep -q '禁止' reference/activity.md
grep -q '我的服务' reference/web3-services.md
grep -q '代付' reference/workflows.md
grep -q '整读' reference/pitfalls.md
```

Expected: exit 0。

- [ ] **Step 7: Commit**

```bash
git add reference/docs.md reference/workflows.md reference/web3-services.md reference/activity.md reference/pitfalls.md
git commit -m "$(cat <<'EOF'
docs(skill): add reference docs, workflows, and product boundaries

Cover verified doc links, mint workflows, Web3 purchase/config, and Activity placeholder.
EOF
)"
```

---

### Task 4: 编写 `examples/`

**Files:**
- Create: `examples/env.md`
- Create: `examples/easy-mint.md`
- Create: `examples/deploy-sponsor-mint.md`

**Interfaces:**
- Consumes: `reference/api.md` 路径与鉴权约定
- Produces: SKILL.md 可引用的可运行示意（curl + 环境变量）

- [ ] **Step 1: 写入 `examples/env.md`**

用下面全文创建文件（内含 bash 代码块）：

````markdown
# 环境变量

```bash
export NFTRAINBOW_APP_ID="YOUR_APP_ID"
export NFTRAINBOW_APP_SECRET="YOUR_APP_SECRET"
export NFTRAINBOW_BASE_URL="https://api.nftrainbow.cn"
```

- 从 Rainbow 开发者控制台创建应用后获取 `app_id` / `app_secret`。
- **禁止**把真实值写入仓库、示例或 commit。
````

- [ ] **Step 2: 写入 `examples/easy-mint.md`**

````markdown
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
````

- [ ] **Step 3: 写入 `examples/deploy-sponsor-mint.md`**

````markdown
# Deploy → Sponsor → Custom Mint

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
````

- [ ] **Step 4: 验收 examples**

```bash
grep -q 'NFTRAINBOW_APP_ID' examples/env.md
grep -q 'mints/easy/urls' examples/easy-mint.md
grep -q 'v1/contracts' examples/deploy-sponsor-mint.md
grep -q 'sponsor' examples/deploy-sponsor-mint.md
! grep -E 'app_secret.:.[^Y"]' examples/*.md
```

Expected: exit 0；示例中无真实 secret 形态（仅 `YOUR_*` 或环境变量展开）。

- [ ] **Step 5: Commit**

```bash
git add examples/env.md examples/easy-mint.md examples/deploy-sponsor-mint.md
git commit -m "$(cat <<'EOF'
docs(skill): add env and mint integration examples

Provide curl-based Easy Mint and deploy-sponsor-mint flows using env vars.
EOF
)"
```

---

### Task 5: 编写根目录 `SKILL.md`

**Files:**
- Create: `SKILL.md`

**Interfaces:**
- Consumes: 全部 `reference/*` 与 `examples/*`
- Produces: agent 入口（name=`nftrainbow`）

- [ ] **Step 1: 写入 `SKILL.md`**

```markdown
---
name: nftrainbow
description: >
  NFTRainbow（NFT 彩虹桥）官方文档与 Open API 集成助手。覆盖铸造工具文档导航、
  合约部署/代付/元数据/铸造/转账/销毁、Web3 Services 购买与配置、以及 Activity 占位说明。
  在用户提及 NFTRainbow、Rainbow API、NFT 铸造、合约代付、元数据、api.nftrainbow.cn、
  docs.nftrainbow.xyz、Web3 Services 或 Activity/POAP 时使用。
---

# NFTRainbow

同时服务：**文档导航**与 **Open API 集成**。中文说明；API 标识符保持英文。

## 读文件规则

1. 先用下方决策树分流。
2. 文档 → [reference/docs.md](reference/docs.md)；必要时拉 GitBook `.md` 或 `?ask=`。
3. 写集成 → [reference/api.md](reference/api.md) + [reference/workflows.md](reference/workflows.md) + [examples/](examples/)。
4. 字段细节 → 对 [openapi/swagger-2.0.json](openapi/swagger-2.0.json) **局部**查找；**禁止默认整读**。
5. Web3 Services → [reference/web3-services.md](reference/web3-services.md)（仅购买/配置）。
6. Activity → [reference/activity.md](reference/activity.md)（占位，不实现）。
7. 坑与硬性规则 → [reference/pitfalls.md](reference/pitfalls.md)。

## 决策树

```
文档 / 概念 / 链接？     → reference/docs.md
要写集成代码？           → api.md + workflows.md + examples/
Web3 Services？          → web3-services.md（禁止展开 RPC/SCAN 调用）
Activity / POAP？        → activity.md（禁止臆造 API）
Conflux 链基础概念？     → 可建议 conflux-docs；Rainbow 代付步骤仍用本 skill
```

## API 要点

- Base：`https://api.nftrainbow.cn`
- Login：`POST /v1/login`（`app_id` + `app_secret`）→ `Authorization: Bearer <token>`
- 环境变量：`NFTRAINBOW_APP_ID` / `NFTRAINBOW_APP_SECRET`（见 [examples/env.md](examples/env.md)）
- 链：`conflux` / `conflux_test`
- 任务 `status`：0 pending / 1 success / 2 failed
- 铸造前确保代付/自动代付就绪
- Pin 默认可信；接口不符时核对线上 swagger 并提示更新 pin 与 `api.md` 同步信息

## 推荐路径

- 最快试铸 → Easy Mint（[examples/easy-mint.md](examples/easy-mint.md)）
- 自有合约 → Deploy → Sponsor → Mint（[examples/deploy-sponsor-mint.md](examples/deploy-sponsor-mint.md)）

## 禁止

- 臆造文档 URL 或 Activity API
- 推荐 Anyweb（用晒啦）
- 把秘密写入仓库
- 在 Web3 Services 场景输出 RPC/SCAN 调用教程
- 默认整读 swagger
```

- [ ] **Step 2: 验收 SKILL.md**

```bash
test -f SKILL.md
grep -q '^name: nftrainbow' SKILL.md
grep -q 'description:' SKILL.md
grep -q 'reference/docs.md' SKILL.md
grep -q '禁止默认整读' SKILL.md
LINES=$(wc -l < SKILL.md)
echo "lines=$LINES"
test "$LINES" -lt 500
```

Expected: `lines=` 远小于 500；全部 grep 成功。

- [ ] **Step 3: Commit**

```bash
git add SKILL.md
git commit -m "$(cat <<'EOF'
feat(skill): add nftrainbow SKILL.md entrypoint

Wire decision tree, progressive disclosure, and hard safety rules for agents.
EOF
)"
```

---

### Task 6: 成品验收（对照 spec §6）

**Files:**
- Modify: none required（若发现缺口则回到对应 Task 修文件并另开 commit）

**Interfaces:**
- Consumes: 全部 skill 产物
- Produces: 验收记录（在 PR/对话中报告；不必新建报告文件除非失败）

- [ ] **Step 1: 文件齐套**

```bash
test -f SKILL.md
test -f openapi/swagger-2.0.json
test -f reference/api.md
test -f reference/docs.md
test -f reference/workflows.md
test -f reference/web3-services.md
test -f reference/activity.md
test -f reference/pitfalls.md
test -f examples/env.md
test -f examples/easy-mint.md
test -f examples/deploy-sponsor-mint.md
```

- [ ] **Step 2: 场景对照检查（人工读 + grep）**

| 场景 | 检查命令 / 期望 |
|------|-----------------|
| 两分钟铸 NFT | `grep -n 'Easy Mint' SKILL.md reference/workflows.md examples/easy-mint.md` → 有路径与文档倾向 |
| 部署再铸造 | `grep -n '代付\|sponsor' reference/workflows.md examples/deploy-sponsor-mint.md` → 含代付，不跳步 |
| Web3 购买配置 | `grep -n '我的服务\|购买' reference/web3-services.md` 且 `! grep -niE 'eth_call|getLogs' reference/web3-services.md` |
| Activity 占位 | `grep -n '禁止' reference/activity.md` |
| 请求体不臆造 | `grep -n '局部\|pin' reference/api.md SKILL.md` |

- [ ] **Step 3: 一层引用与术语**

```bash
# SKILL 只应链到 reference/ 或 examples/ 或 openapi/，不链 reference/foo/bar
! grep -E '\]\(reference/.+/.+\)' SKILL.md
grep -q '晒啦' reference/docs.md reference/activity.md
grep -q 'Anyweb' reference/docs.md reference/pitfalls.md
```

- [ ] **Step 4: 若全部通过，提交验收用的小幅修正（仅当有修正时）**

无修正则跳过 commit。若有修正：

```bash
git add -u
git commit -m "$(cat <<'EOF'
fix(skill): address acceptance checklist gaps

Align wording and links with the design spec verification scenarios.
EOF
)"
```

- [ ] **Step 5: 更新 design spec 状态行（可选但推荐）**

将 `docs/superpowers/specs/2026-07-16-nftrainbow-skill-design.md` 顶部 `状态：待用户审阅` 改为 `状态：已批准并实现中` 或实现全部完成后改为 `状态：已实现`。

```bash
git add docs/superpowers/specs/2026-07-16-nftrainbow-skill-design.md
git commit -m "docs: mark nftrainbow design spec approved"
```

---

## Self-Review (plan author)

| Spec 项 | 对应 Task |
|---------|-----------|
| pin swagger | Task 1 |
| api.md + 同步元数据 | Task 2 |
| docs/workflows/web3/activity/pitfalls | Task 3 |
| examples | Task 4 |
| SKILL.md | Task 5 |
| §6 验收 | Task 6 |
| 中文 / 凭证 / 禁止整读 / Activity 占位 / Web3 范围 | Global + Tasks 2–5 |
| 不写 sync 脚本 | 未列入（符合 spec） |

Placeholder scan: 无 TBD；示例中注明删除混乱 login 片段。Accounts 收录理由：铸造到手机号/托管账户辅助，仍属铸造主链路支持面。
