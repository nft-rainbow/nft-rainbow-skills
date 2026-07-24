# Setup（KYC + App 凭证）

获取 Open API 所需的 `app_id` / `app_secret`，并准备约定路径的 `.env`。

## 1. 完成 KYC（用户操作）

1. 打开 [Rainbow 控制台](https://console.nftrainbow.cn/)
2. 注册并登录
3. 提交实名信息，等待审核

在控制台判断审核状态：

| 控制台表现 | 含义 |
|------------|------|
| 页面顶部显示：`请耐心等待审核通过，我们会在2-3天完成审核。` | 未通过 / 审核中 |
| **不显示**上述提示 | 已通过 |

更新认证信息：主页右上角用户名 → 用户设置 → 认证信息 → 更新。

**无** KYC 查询/提交 Open API；勿臆造 endpoint。问「怎么查 KYC 状态」→ 只能看上表控制台提示。

`POST /v1/login` 等失败若出现 `KYC required` 或错误码 `40105` → 回控制台完成实名，用上表判断是否通过。错误码：https://docs.nftrainbow.xyz/about-the-apis/error-codes

## 2. 创建 App（用户操作）

审核通过后，在“我的项目”中创建项目，点击“查看AppKey”，取得 `app_id` 与 `app_secret`。

## 3. 准备 `.env`

Agent：

1. 用户已指定 `.env` 路径时使用该路径，否则默认使用当前项目的 `.env`；无需为默认路径询问用户。
2. 使用默认路径时，确保当前项目的 `.gitignore` 包含一行 `.env`；不存在则创建。自定义路径位于当前项目内时，将其项目相对路径加入 `.gitignore`。
3. 约定位置没有 `.env` 时创建以下模板；已有则不覆盖：

```dotenv
NFTRAINBOW_APP_ID=
NFTRAINBOW_APP_SECRET=
NFTRAINBOW_BASE_URL=https://api.nftrainbow.cn
```

4. 告知用户编辑该文件，填入控制台取得的 `app_id` 与 `app_secret`；空值时等待用户填好，勿进入鉴权。

**Setup 完成条件**：约定位置存在 `.env`，且 `NFTRAINBOW_APP_ID` 与 `NFTRAINBOW_APP_SECRET` 均非空。

## Agent 行为

- 未声明「KYC/实名已通过」时，先复述 §1（或指向本页），再进 `auth.md` 或 `activity.md`
- 已声明「KYC/实名已通过」→ 可跳过 §1 展开；可一句提醒用控制台顶部提示自检

## 官方参考

- https://docs.nftrainbow.xyz/docs/faqs
- https://docs.nftrainbow.xyz/tutorials/interactive
- https://docs.nftrainbow.xyz/tutorials/mints/mints-zh
- https://docs.nftrainbow.xyz/about-the-apis/error-codes
