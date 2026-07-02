# 方法

所有方法均通过 **HTTP POST** 调用 `https://<your-nexlink-api-host>/bot<token>/<method>`。请求与响应格式参见[发起请求](README.md#发起请求)。

- [getMe](#getme)
- [getUpdates](#getupdates)
- [sendMessage](#sendmessage)
- [sendPhoto](#sendphoto)
- [sendVideo](#sendvideo)
- [sendDocument](#senddocument)
- [sendAudio](#sendaudio)
- [sendVoice](#sendvoice)
- [sendLocation](#sendlocation)
- [forwardMessage](#forwardmessage)
- [editMessageText](#editmessagetext)
- [deleteMessage](#deletemessage)
- [answerCallbackQuery](#answercallbackquery)
- [setMyCommands](#setmycommands)
- [getMyCommands](#getmycommands)
- [deleteMyCommands](#deletemycommands)
- [setWebhook](#setwebhook)
- [deleteWebhook](#deletewebhook)
- [getWebhookInfo](#getwebhookinfo)

---

## getMe

返回机器人的基本信息。

**参数：** 无

**返回：** [User](TYPES.md#user)

```bash
POST /bot<token>/getMe
```

**响应：**

```json
{
  "ok": true,
  "result": {
    "id": "bot_a1b2c3d4",
    "is_bot": true,
    "first_name": "Weather Bot",
    "username": "weatherbot"
  }
}
```

---

## getUpdates

[Webhook](WEBHOOKS.md) 的长轮询替代方案。返回机器人待处理的 [Update](TYPES.md#update) 对象。无论是否配置 Webhook，服务端都会为每个机器人保存更新，因此即使不设置 Webhook 也能轮询。

**参数：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `offset` | Integer | 否 | 返回 `update_id >= offset` 的更新。传入 offset 同时会**确认**（删除）所有 `update_id` 更小的更新，使其不再重复返回。应设为 `last_update_id + 1`。 |
| `limit` | Integer | 否 | 返回更新的最大数量，1-100。默认 100。 |
| `timeout` | Integer | 否 | 长轮询超时秒数，0-50。`0` 表示立即返回（短轮询）。若没有待处理更新，请求最多等待 `timeout` 秒直到有更新到达。默认 0。 |

**返回：** [Update](TYPES.md#update) 数组。若在超时前无更新到达，则返回空数组。

```bash
POST /bot<token>/getUpdates
Content-Type: application/json

{
  "offset": 100002,
  "limit": 100,
  "timeout": 30
}
```

**响应：**

```json
{
  "ok": true,
  "result": [
    {
      "update_id": 100002,
      "message": {
        "message_id": "srv_msg_a1b2c3d4",
        "from": {"id": "user_abc123", "is_bot": false, "first_name": "Alice"},
        "chat": {"id": "user_abc123", "type": "private"},
        "date": 1719846000,
        "text": "hello"
      }
    }
  ]
}
```

> **轮询循环：** 循环调用 `getUpdates`。每次响应后记录最大的 `update_id`，下次调用时传入 `offset = 最大值 + 1`，即可确认本批更新并只获取更新的部分。

---

## sendMessage

发送一条文本消息。

**参数：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `chat_id` | String | 是 | 目标会话的唯一标识（OpenIM userID 或 groupID） |
| `text` | String | 是 | 消息文本，1-4096 个字符 |
| `reply_markup` | [InlineKeyboardMarkup](TYPES.md#inlinekeyboardmarkup) | 否 | 附加到消息的内联键盘 |

**返回：** [Message](TYPES.md#message)

```bash
POST /bot<token>/sendMessage
Content-Type: application/json

{
  "chat_id": "user_abc123",
  "text": "Hello! How can I help you?",
  "reply_markup": {
    "inline_keyboard": [
      [
        {"text": "Help", "callback_data": "help"},
        {"text": "Settings", "callback_data": "settings"}
      ]
    ]
  }
}
```

---

> **通用媒体字段。** 除下方各方法列出的参数外，所有媒体发送方法（`sendPhoto`、`sendVideo`、`sendDocument`、`sendAudio`、`sendVoice`）都接受以下可选字段。它们会被透传到消息中，便于客户端渲染更丰富的预览：
>
> | 字段 | 类型 | 说明 |
> |------|------|------|
> | `mime_type` | String | 文件的 MIME 类型 |
> | `file_size` | Integer | 文件大小（字节） |
> | `file_name` | String | 原始文件名（文档） |
> | `width` | Integer | 媒体宽度，像素（图片/视频） |
> | `height` | Integer | 媒体高度，像素（图片/视频） |
> | `duration` | Integer | 时长，秒（音频/语音/视频） |
> | `thumb_url` | String | 缩略图 URL（视频） |
> | `thumb_width` | Integer | 缩略图宽度，像素 |
> | `thumb_height` | Integer | 缩略图高度，像素 |

## sendPhoto

发送一张图片。

**参数：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `chat_id` | String | 是 | 目标会话 ID |
| `url` | String | 是 | 图片 URL |
| `caption` | String | 否 | 图片说明，0-1024 个字符 |
| `reply_markup` | [InlineKeyboardMarkup](TYPES.md#inlinekeyboardmarkup) | 否 | 内联键盘 |

**返回：** [Message](TYPES.md#message)

---

## sendVideo

发送一段视频。

**参数：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `chat_id` | String | 是 | 目标会话 ID |
| `url` | String | 是 | 视频 URL |
| `caption` | String | 否 | 视频说明 |
| `reply_markup` | [InlineKeyboardMarkup](TYPES.md#inlinekeyboardmarkup) | 否 | 内联键盘 |

**返回：** [Message](TYPES.md#message)

---

## sendDocument

发送一个通用文件（文档）。

**参数：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `chat_id` | String | 是 | 目标会话 ID |
| `url` | String | 是 | 文档 URL |
| `caption` | String | 否 | 文档说明 |
| `reply_markup` | [InlineKeyboardMarkup](TYPES.md#inlinekeyboardmarkup) | 否 | 内联键盘 |

**返回：** [Message](TYPES.md#message)

---

## sendAudio

发送一个音频文件。

**参数：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `chat_id` | String | 是 | 目标会话 ID |
| `url` | String | 是 | 音频 URL |
| `caption` | String | 否 | 音频说明 |
| `reply_markup` | [InlineKeyboardMarkup](TYPES.md#inlinekeyboardmarkup) | 否 | 内联键盘 |

**返回：** [Message](TYPES.md#message)

---

## sendVoice

发送一条语音消息。

**参数：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `chat_id` | String | 是 | 目标会话 ID |
| `url` | String | 是 | 语音录音的 URL |
| `caption` | String | 否 | 语音说明 |
| `reply_markup` | [InlineKeyboardMarkup](TYPES.md#inlinekeyboardmarkup) | 否 | 内联键盘 |

**返回：** [Message](TYPES.md#message)

---

## sendLocation

发送地图上的一个坐标点。

**参数：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `chat_id` | String | 是 | 目标会话 ID |
| `latitude` | Float | 是 | 位置纬度 |
| `longitude` | Float | 是 | 位置经度 |
| `reply_markup` | [InlineKeyboardMarkup](TYPES.md#inlinekeyboardmarkup) | 否 | 内联键盘 |

**返回：** [Message](TYPES.md#message)

---

## forwardMessage

将机器人此前发送或接收过的某条消息内容复制到另一个会话。采用复制语义（不带"转发自"头部），与 Telegram 行为一致。只有机器人有记录的消息才能被转发。

**参数：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `chat_id` | String | 是 | 要复制到的目标会话 |
| `message_id` | String | 是 | 机器人已知消息的 `message_id`（它发送或接收过的消息） |
| `from_chat_id` | String | 否 | 来源会话 ID（保留字段；消息通过 `message_id` 定位） |

**返回：** [Message](TYPES.md#message)

> 若 `message_id` 不为机器人所知，返回 `400 Bad Request: message to forward not found`。

---

## editMessageText

编辑机器人已发送消息的文本。OpenIM 不支持原地编辑，因此会撤回原消息并发送一条替代消息 —— **返回的消息带有新的 `message_id`**，你应将其保存为当前消息。

**参数：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `chat_id` | String | 否 | 目标会话（仅供参考） |
| `message_id` | String | 是 | 机器人自己要编辑的消息的 `message_id` |
| `text` | String | 是 | 新文本，1-4096 个字符 |
| `reply_markup` | [InlineKeyboardMarkup](TYPES.md#inlinekeyboardmarkup) | 否 | 新的内联键盘 |

**返回：** [Message](TYPES.md#message) —— 带有**新的** `message_id`。

> 错误：`400 Bad Request: message not found`，或 `400 Bad Request: can only edit the bot's own messages`。

---

## deleteMessage

删除（撤回）机器人已发送的一条消息。

**参数：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `chat_id` | String | 否 | 目标会话（仅供参考） |
| `message_id` | String | 是 | 机器人已知消息的 `message_id` |

**返回：** 成功时返回 `true`。

> 若 `message_id` 不为机器人所知，返回 `400 Bad Request: message not found`。

---

## answerCallbackQuery

对来自[内联键盘](TYPES.md#inlinekeyboardbutton)的回调查询作出应答。应答会以聊天界面顶部的通知或弹窗形式展示给用户。

**参数：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `callback_query_id` | String | 是 | 要应答的查询的唯一标识 |
| `text` | String | 否 | 通知文本（0-200 个字符） |
| `show_alert` | Boolean | 否 | 若为 `true`，显示弹窗而非顶部通知。默认 `false` |

**返回：** 成功时返回 `true`。

```bash
POST /bot<token>/answerCallbackQuery
Content-Type: application/json

{
  "callback_query_id": "abc-def-123",
  "text": "Processing your order...",
  "show_alert": false
}
```

---

## setMyCommands

设置机器人的命令列表。用户在聊天中输入 `/` 时可看到这些命令。

**参数：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `commands` | [BotCommand](TYPES.md#botcommand) 数组 | 是 | 要设置的命令列表。最多 100 条。 |

**返回：** 成功时返回 `true`。

```bash
POST /bot<token>/setMyCommands
Content-Type: application/json

{
  "commands": [
    {"command": "start", "description": "Start the bot"},
    {"command": "help", "description": "Show available commands"},
    {"command": "settings", "description": "Change bot settings"}
  ]
}
```

---

## getMyCommands

获取机器人当前的命令列表。

**参数：** 无

**返回：** [BotCommand](TYPES.md#botcommand) 数组

---

## deleteMyCommands

删除机器人的命令列表。

**参数：** 无

**返回：** 成功时返回 `true`。

---

## setWebhook

指定一个 URL，NexLink 会向其 POST [Update](TYPES.md#update) 对象。

**参数：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `url` | String | 是 | 接收更新的 HTTPS URL |
| `secret_token` | String | 否 | 密钥令牌（1-256 个字符），会在每个 Webhook 请求的 `X-NexLink-Bot-Secret-Token` 头中发送 |
| `allowed_updates` | String 数组 | 否 | 要接收的更新类型列表，例如 `["message", "callback_query"]`。默认接收全部。 |

**返回：** 成功时返回 `true`。

```bash
POST /bot<token>/setWebhook
Content-Type: application/json

{
  "url": "https://myserver.com/bot/webhook",
  "secret_token": "mysecret123",
  "allowed_updates": ["message", "callback_query"]
}
```

载荷格式、签名验证与重试策略参见 [Webhooks](WEBHOOKS.md)。

---

## deleteWebhook

移除 Webhook 集成。

**参数：** 无

**返回：** 成功时返回 `true`。

---

## getWebhookInfo

返回当前 Webhook 状态。

**参数：** 无

**返回：** [WebhookInfo](TYPES.md#webhookinfo)
