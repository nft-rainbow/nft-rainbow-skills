# Rainbow Activity（含 POAP）

项目方通过控制台发行活动纪念 / 奖章类 NFT（POAP 也属于一种 Activity），并提供统一领取页；用户用晒啦（手机号登录）领取。

**官方文档：** https://docs.nftrainbow.xyz/tutorials/guides/poap  
**控制台：** https://console.nftrainbow.cn

## 与铸造 Open API 的边界

- Activity **走控制台 + 领取页**，不是 `/v1/mints` 那套 Open API 集成路径。
- **不要编造** Activity 专用 Open API、领取接口或晒啦 SDK 细节。
- 合约部署 / 代付可与铸造工具共用（控制台或 Open API 部署均可）；活动创建与发放链接以控制台为准。

## 准备工作（项目方）

创建活动前完成：

1. 注册并登录 [NFTRainbow 控制台](https://console.nftrainbow.cn)
2. 完成 KYC（实名认证），见 [kyc.md](kyc.md)
3. 创建项目并部署合约
4. 为合约设置代付（需先充值）。单个 NFT 存储约 `0.6–0.7 CFX`，按发行量预留（例如 100 个约 `70 CFX`）。详见：https://docs.nftrainbow.xyz/tutorials/guides/kong-zhi-tai-he-yue-dai-fu-she-zhi

## 项目方：创建活动并发放链接

1. 在 NFT 活动页面，点击 `创建活动/创建POAP`
2. 填写活动详情（名称、说明、徽章图片等）
3. 点击 `管理藏品`
4. 绑定已部署的合约（**合约需已完成代付**）
5. 保存后返回活动列表，打开活动链接（领取页），发给用户

## 用户：领取 NFT

1. 打开活动领取链接
2. 点击 `连接钱包`，输入**手机号码**登录
3. 授权账户
4. 确认钱包已连接后，点击 `领取`
5. 等待数十秒，完成后可去钱包查看
6. 登录**晒啦**钱包查看已领 NFT（不要推荐 Anyweb）

## Agent 应答要点

- 问「怎么发 POAP / Activity」→ 先按 [kyc.md](kyc.md) 完成 KYC 卡控（未声明已通过则先复述 KYC）；再按上文控制台步骤说明，并给出官方文档链接
- 问「有没有 Activity API」→ 说明当前文档为控制台流程；勿臆造 endpoint
- 问「用户怎么领」→ 手机号登录领取页 → 晒啦查看（领取方用户无需项目方 KYC）
