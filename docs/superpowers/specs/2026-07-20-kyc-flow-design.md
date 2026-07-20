# KYC Flow — Design Spec

## Goal

Agent 在指导 mint / Open API 集成 / Rainbow Activity 前，必须先完成控制台 KYC（实名）前置；KYC 步骤、状态判断与更新路径有单一真相源；明确**无** KYC 查询 Open API。

## Decisions (locked)

| 项 | 选择 |
|----|------|
| 覆盖范围 | Open API + Activity，统一真相源 |
| Agent 卡控 | 硬前置：未声明已通过时先复述 KYC |
| 内容粒度 | 官方/已确认事实；不臆造表单字段、证件类型 |
| 文档结构 | 新建 `reference/kyc.md`；其余短引用 |
| 状态查询 | 仅控制台；无查询 API |

## Scope

### Create

- `nftrainbow/reference/kyc.md`

### Modify (short refs + order only; no long duplicated KYC copy)

- `nftrainbow/SKILL.md` — 读文件规则、决策树、API 要点：先 KYC 再 auth
- `nftrainbow/reference/auth.md` — 开头声明须先完成 KYC
- `nftrainbow/reference/activity.md` — 准备步骤引用 `kyc.md`
- `nftrainbow/reference/workflows.md`
- `nftrainbow/examples/easy-mint.md`
- `nftrainbow/examples/deploy-sponsor-mint.md`
- `nftrainbow/examples/env.md`
- `nftrainbow/reference/api.md`
- `nftrainbow/reference/pitfalls.md` — KYC 硬规则 + `40105`
- `nftrainbow/reference/docs.md` — 索引官方 FAQs / interactive / error-codes + 控制台
- `README.md` — 目录树或 Features 中提及 KYC 前置（短）

### Out of Scope

- 编造 KYC 查询/提交 Open API path
- 臆造实名表单字段、证件类型、审核 SLA 细节（除用户确认的顶部提示文案与「2-3 天」）
- 更新 swagger pin
- SDK / 控制台截图仓库化

## `kyc.md` content

1. **为何需要** — 审核通过后才能创建 App、调 Open API、发 Activity（对齐 FAQ、interactive 流程图）
2. **提交实名** — [控制台](https://console.nftrainbow.cn/) → 注册/登录 → 提交实名 → 等待审核
3. **状态判断（仅控制台）**
   - 页面顶部显示：`请耐心等待审核通过，我们会在2-3天完成审核。` → 未通过 / 审核中
   - **不显示**该提示 → 已通过
   - 明确：无 KYC 查询 Open API
4. **更新认证信息** — 首页 → 右上角用户名 → 用户设置 → 认证信息 → 更新
5. **API 侧信号** — login 等失败出现 `KYC required` / 错误码 `40105` → 回控制台完成实名；链接 [Error codes](https://docs.nftrainbow.xyz/about-the-apis/error-codes)
6. **官方参考** — FAQs、interactive、mints-zh 准备工作

## Agent behavior

| 场景 | 行为 |
|------|------|
| 问 mint / 调 `/v1/*` / 发 Activity | 先复述 KYC（或指向 `kyc.md`），再进 auth / Activity |
| 用户已声明「KYC/实名已通过」 | 可跳过展开；可一句提醒控制台顶部提示判据 |
| `KYC required` / `40105` | 引导控制台完成审核；勿编造查询 API |
| 问「怎么查 KYC 状态」 | 只能看控制台顶部提示，无 Open API |

## Wiring order

- **集成 / mint：** `kyc.md` → `auth.md`（创建 App → login → Bearer）→ workflows / examples
- **Activity：** `kyc.md` → 创建项目/合约/代付 → 创建活动（`activity.md` 其余步骤）
- **`SKILL.md` 决策树：** `KYC / 实名 / 审核？ → reference/kyc.md`；写集成：`先 kyc.md，再 auth.md，再 api/workflows/examples`

## Acceptance Criteria

- [x] 存在 `nftrainbow/reference/kyc.md`，含：控制台 URL、顶部提示文案、更新路径、无查询 API、`40105`
- [x] `SKILL.md` 强制集成/Activity 相关路径先读 KYC
- [x] `auth.md` / `activity.md` / workflows / 两个 mint 示例有 KYC 引用或等价前置
- [x] 其余文件仅短引用，无互相矛盾的长文副本
- [x] 仓库内无臆造的 KYC Open API path

## Verify

```sh
test -f nftrainbow/reference/kyc.md
rg -n 'kyc\.md|实名|KYC required|40105|请耐心等待审核' nftrainbow/
```

## Official references

| 主题 | URL |
|------|-----|
| 控制台 | https://console.nftrainbow.cn/ |
| FAQs（实名 / KYC required） | https://docs.nftrainbow.xyz/docs/faqs |
| Interactive（注册 → KYC → 创建 APP） | https://docs.nftrainbow.xyz/tutorials/interactive |
| 铸造准备（实名认证） | https://docs.nftrainbow.xyz/tutorials/mints/mints-zh |
| Error codes（40105） | https://docs.nftrainbow.xyz/about-the-apis/error-codes |
