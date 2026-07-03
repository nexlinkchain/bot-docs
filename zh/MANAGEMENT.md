# 管理与发现

用于创建、配置和查找机器人的面向用户的 API。它们与 [Bot API 方法](METHODS.md)（`/bot<token>/*`）不同 —— 这些接口由 NexLink App 代表已登录用户调用，而非由机器人自身调用。

- [机器人管理 API](#机器人管理-api)
- [机器人发现 API](#机器人发现-api)

---

## 机器人管理 API

面向用户的机器人创建与管理 API。所有端点均使用 `POST`，并需要 `CheckNexlinkToken` 鉴权头（NexLink 用户会话）。

**基础路径：** `/bot_mgmt`

### Create Bot

创建一个新机器人。返回机器人信息和一个**令牌**（仅显示一次）。

```
POST /bot_mgmt/create
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `username` | String | 是 | 唯一句柄，1-32 个字符（a-z、0-9、下划线） |
| `displayName` | String | 是 | 显示名 |
| `about` | String | 否 | 简短描述（最多 512 个字符） |
| `avatarUrl` | String | 否 | 头像图片 URL |

**响应：**

```json
{
  "errCode": 0,
  "data": {
    "bot": {
      "botId": "bot_a1b2c3d4",
      "username": "weatherbot",
      "displayName": "Weather Bot",
      "about": "Get weather forecasts",
      "avatarUrl": "https://...",
      "webhookEnabled": false,
      "isPublished": false,
      "status": 1,
      "createdAt": 1719846000
    },
    "token": "bot_a1b2c3d4:c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8"
  }
}
```

> **重要：** `token` 仅在创建时返回一次，请妥善保存。

---

### Update Bot

更新机器人元数据。仅机器人所有者可更新。

```
POST /bot_mgmt/update
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `botId` | String | 是 | 机器人 ID |
| `displayName` | String | 否 | 新的显示名 |
| `about` | String | 否 | 新的简介 |
| `description` | String | 否 | 更长的描述（最多 2048 个字符） |
| `avatarUrl` | String | 否 | 新的头像 URL |

---

### Delete Bot

软删除一个机器人。仅机器人所有者可删除。

```
POST /bot_mgmt/delete
```

| 字段 | 类型 | 必填 |
|------|------|------|
| `botId` | String | 是 |

---

### List My Bots

列出当前已鉴权用户拥有的所有机器人。

```
POST /bot_mgmt/my_bots
```

**请求：** `{}`

**响应：** 机器人对象数组。

---

### Regenerate Token

作废当前令牌并返回一个新令牌。

```
POST /bot_mgmt/regenerate_token
```

| 字段 | 类型 | 必填 |
|------|------|------|
| `botId` | String | 是 |

**响应：**

```json
{
  "errCode": 0,
  "data": {
    "token": "bot_a1b2c3d4:new_random_40_hex_characters_here_1234"
  }
}
```

---

### Set Webhook（管理端）

从 NexLink App UI 配置 Webhook 投递。

```
POST /bot_mgmt/set_webhook
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `botId` | String | 是 | 机器人 ID |
| `url` | String | 是 | Webhook URL（HTTPS） |
| `secret` | String | 否 | 用于签名验证的 Webhook 密钥 |
| `allowedUpdates` | String 数组 | 否 | `["message", "callback_query"]` |

---

### Delete Webhook（管理端）

禁用 Webhook 投递。

```
POST /bot_mgmt/delete_webhook
```

| 字段 | 类型 | 必填 |
|------|------|------|
| `botId` | String | 是 |

---

### Get Webhook Info（管理端）

返回 Webhook 配置与状态。

```
POST /bot_mgmt/get_webhook_info
```

| 字段 | 类型 | 必填 |
|------|------|------|
| `botId` | String | 是 |

**响应：**

```json
{
  "errCode": 0,
  "data": {
    "url": "https://myserver.com/webhook",
    "enabled": true,
    "secret": true,
    "lastError": "",
    "lastErrorAt": 0,
    "allowedUpdates": ["message", "callback_query"]
  }
}
```

---

### Publish Bot

使机器人在公共机器人商店中可见。

```
POST /bot_mgmt/publish
```

| 字段 | 类型 | 必填 |
|------|------|------|
| `botId` | String | 是 |

---

### Unpublish Bot

将机器人从公共机器人商店移除。为 [Publish Bot](#publish-bot) 的逆操作。

```
POST /bot_mgmt/unpublish
```

| 字段 | 类型 | 必填 |
|------|------|------|
| `botId` | String | 是 |

---

### Submit Callback Query

当用户点击内联键盘按钮时，由 NexLink Flutter 客户端调用。创建一条回调查询记录并将其排入 Webhook 投递队列。

```
POST /bot_mgmt/callback_query
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `botId` | String | 是 | 发送该键盘的机器人 |
| `chatId` | String | 是 | OpenIM userID 或 groupID |
| `messageId` | String | 是 | 带键盘消息的 OpenIM serverMsgID |
| `data` | String | 是 | 按钮的 `callback_data` 值 |

**响应：**

```json
{
  "errCode": 0,
  "data": {
    "queryId": "cbq_e5f6a7b8"
  }
}
```

---

## 机器人发现 API

用于查找机器人的公共端点。均使用 `POST` 并需要 `CheckNexlinkToken` 鉴权。

**基础路径：** `/bots`

### Search Bots

按关键词搜索已发布的机器人。

```
POST /bots/search
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `keyword` | String | 否 | 搜索关键词 |
| `offset` | Integer | 否 | 分页偏移量（默认：0） |
| `limit` | Integer | 否 | 每页结果数（默认：20，最大：100） |

---

### Featured Bots

返回精选/推荐的已发布机器人。

```
POST /bots/featured
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `limit` | Integer | 否 | 最大结果数（默认：20，最大：50） |

---

### Bot Profile

按 ID 或用户名获取机器人的公开资料。

```
POST /bots/profile
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `botId` | String | 否 | 机器人 ID（`botId` 与 `username` 二选一必填） |
| `username` | String | 否 | 机器人用户名（`botId` 与 `username` 二选一必填） |

---

### Bot Commands

获取机器人的公开命令列表。

```
POST /bots/commands
```

| 字段 | 类型 | 必填 |
|------|------|------|
| `botId` | String | 是 |

**响应：**

```json
{
  "errCode": 0,
  "data": [
    {"command": "start", "description": "Start the bot"},
    {"command": "help", "description": "Show help"}
  ]
}
```

---

## 深度链接

每个机器人都可以通过 URL 直达——相当于 Telegram 的 `t.me/bot?start=` 链接：

```
https://{web-domain}/bot/{bot_id}?start={payload}
nexlink://bot?id={bot_id}&start={payload}
```

打开任一链接会直接进入机器人聊天页。当链接带有 `start` 参数时，客户端会显示 **Start** 按钮（即使聊天已有历史记录）；用户点击后发送 `/start {payload}`——payload 仅在用户确认后才会送达，与 Telegram 行为一致。

机器人以普通文本消息收到它：

```json
{"message": {"text": "/start ref_711", "from": {"id": "5168018091"}, ...}}
```

payload 可用于推荐来源追踪、账号绑定（放入一次性 token）或跳转到特定内容。payload 应使用 URL 安全字符，建议不超过 64 个字符。
