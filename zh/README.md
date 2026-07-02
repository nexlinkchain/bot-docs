# NexLink 机器人 API（Bot API）

> 用于在 NexLink 上构建机器人的 HTTP API。

机器人是通过程序控制的特殊 OpenIM 账号。它们可以接收消息、发送回复、展示[内联键盘](KEYBOARDS.md)，并通过 [Webhook](WEBHOOKS.md) 进行交互。

---

## 快速开始

### 1. 创建机器人

使用[机器人管理 API](MANAGEMENT.md#create-bot) 或 NexLink App 的 **机器人设置** 页面创建机器人。你会获得一个 **令牌（token）**，形如：

```
bot_a1b2c3d4:c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8
```

> **请妥善保管令牌。** 任何持有令牌的人都能控制你的机器人。若令牌泄露，请立即[重新生成](MANAGEMENT.md#regenerate-token)。

### 2. 配置 Webhook

通过[设置 Webhook](METHODS.md#setwebhook) 告诉 NexLink 将更新投递到哪里。每当用户与你的机器人交互时，NexLink 都会向你的 URL POST 一个 JSON 格式的 [Update](TYPES.md#update) 对象。

### 3. 开始接收消息

当用户向机器人发送消息时，NexLink 会通过 Webhook 投递一个 [Message](TYPES.md#message) 对象。你的服务器处理后调用 Bot API 进行回复。

### Webhook 或轮询

接收更新有两种方式，任选其一：

- **Webhook** —— NexLink 将每个 [Update](TYPES.md#update) 推送到你的 HTTPS URL。适合生产环境。参见 [setWebhook](METHODS.md#setwebhook)。
- **轮询（Polling）** —— 你的机器人通过 [getUpdates](METHODS.md#getupdates) 主动拉取待处理的更新。无需公网 URL，非常适合本地开发和测试。

服务端会为**每个**机器人排队保存待处理的更新，因此无论是否配置 Webhook，`getUpdates` 都可用。若你同时启用 Webhook **并且**进行轮询，每条更新会通过两条路径各投递一次 —— 请只选择其中一种，以免重复处理。

---

## 发起请求

所有方法都通过 **HTTP POST** 调用：

```
https://<your-nexlink-api-host>/bot<token>/<method>
```

示例 —— 发送一条文本消息：

```bash
curl -X POST https://api.nexlink.im/botbot_a1b2c3d4:c9d0e1f2.../sendMessage \
  -H "Content-Type: application/json" \
  -d '{"chat_id": "user_abc123", "text": "Hello!"}'
```

### 响应格式

每个响应都是一个 JSON 对象：

```json
{
  "ok": true,
  "result": { ... }
}
```

出错时：

```json
{
  "ok": false,
  "error_code": 400,
  "description": "Bad Request: chat_id is required"
}
```

---

## 机器人鉴权

机器人令牌嵌入在 URL 路径中。调用 Bot API 时无需额外的请求头。

```
/bot<token>/methodName
     ^^^^^^
     你的密钥令牌
```

令牌格式为 `<bot_id>:<secret>`，例如 `bot_a1b2c3d4:random40hex`。服务端仅保存其 SHA-256 哈希。

---

## 文档目录

| 章节 | 说明 |
|---|---|
| [方法](METHODS.md) | 所有 Bot API 方法 —— 发送、编辑、轮询、命令、Webhook 配置 |
| [类型](TYPES.md) | 请求、响应与 Webhook 中使用的对象定义 |
| [内联键盘](KEYBOARDS.md) | 构建内联键盘与处理回调查询 |
| [Webhooks](WEBHOOKS.md) | Webhook 配置、载荷、签名验证、重试策略 |
| [管理与发现](MANAGEMENT.md) | 面向用户的机器人管理与公开发现端点 |
| [错误与参考](ERRORS.md) | 错误码、消息流转、内容类型映射、约束限制 |
