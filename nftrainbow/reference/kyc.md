# KYC 前置（实名认证）

问 mint / 写 `/v1/*` 集成 / 发 Rainbow Activity 时，**必须先完成**本页；再进入鉴权（`auth.md`）或 Activity 创建流程。

## 1. 为何需要

官方审核通过后，才能创建应用（App）、调用 Open API、发行 Activity。未完成实名时，login 等可能返回 `KYC required`。

## 2. 提交实名

1. 打开 [Rainbow 控制台](https://console.nftrainbow.cn/)
2. 注册并登录
3. 提交实名信息，等待审核

## 3. 如何判断状态（仅控制台）

**没有** KYC 查询 Open API；只能在控制台查看：

| 控制台表现 | 含义 |
|------------|------|
| 页面顶部显示：`请耐心等待审核通过，我们会在2-3天完成审核。` | 未通过 / 审核中 |
| **不显示**上述提示 | 已通过 |

## 4. 更新认证信息

首页 → 右上角点击用户名 → 用户设置 → 认证信息 → 更新

## 5. API 侧信号

- 无专用 KYC 查询/提交接口；勿臆造 endpoint
- `POST /v1/login` 等失败若出现 `KYC required` 或错误码 `40105` → 回控制台完成实名，用第 3 节判断是否通过
- 错误码说明：https://docs.nftrainbow.xyz/about-the-apis/error-codes

## 官方参考

| 主题 | URL |
|------|-----|
| FAQs | https://docs.nftrainbow.xyz/docs/faqs |
| Interactive（注册 → KYC → 创建 APP） | https://docs.nftrainbow.xyz/tutorials/interactive |
| 铸造准备 | https://docs.nftrainbow.xyz/tutorials/mints/mints-zh |
| Error codes | https://docs.nftrainbow.xyz/about-the-apis/error-codes |

## Agent 行为

- 用户问「怎么 mint / 调 API / 发 Activity」→ 先复述本节 2–3（或指向本页），再进 `auth.md` 或 `activity.md`
- 用户已声明「KYC/实名已通过」→ 可跳过展开；可一句提醒用控制台顶部提示自检
- 出现 `KYC required` / `40105` → 引导控制台完成审核；勿编造查询 API
- 问「怎么查 KYC 状态」→ 只能看控制台顶部提示，无 Open API
