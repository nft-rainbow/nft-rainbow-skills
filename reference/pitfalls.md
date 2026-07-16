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
