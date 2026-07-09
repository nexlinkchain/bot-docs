# 类型

在 Bot API 中广泛使用的对象定义 —— 出现在方法参数、响应以及 [Webhook](WEBHOOKS.md) 载荷中。

- [User](#user)
- [Chat](#chat)
- [Message](#message)
- [PhotoSize](#photosize)
- [Video](#video)
- [Document](#document)
- [Voice](#voice)
- [Location](#location)
- [MessageEntity](#messageentity)
- [CallbackQuery](#callbackquery)
- [Update](#update)
- [InlineKeyboardMarkup](#inlinekeyboardmarkup)
- [InlineKeyboardButton](#inlinekeyboardbutton)
- [WebAppInfo](#webappinfo)
- [BotCommand](#botcommand)
- [WebhookInfo](#webhookinfo)

---

## User

表示一个 NexLink 用户或机器人。

| 字段 | 类型 | 说明 |
|------|------|------|
| `id` | String | 唯一标识（OpenIM userID） |
| `is_bot` | Boolean | 若为机器人则为 `true` |
| `first_name` | String | 用户或机器人的显示名 |
| `username` | String | *可选。* 用户或机器人的用户名 |

## Chat

表示一个会话。

| 字段 | 类型 | 说明 |
|------|------|------|
| `id` | String | 会话唯一标识（OpenIM userID 或 groupID） |
| `type` | String | 会话类型：`"private"`、`"group"` 或 `"channel"` |

## Message

表示一条消息。

| 字段 | 类型 | 说明 |
|------|------|------|
| `message_id` | String | 消息唯一标识（OpenIM serverMsgID） |
| `from` | [User](#user) | *可选。* 消息发送者 |
| `chat` | [Chat](#chat) | 消息所属会话 |
| `date` | Integer | 消息发送时间（Unix 时间戳） |
| `text` | String | *可选。* 消息文本内容 |
| `caption` | String | *可选。* 媒体消息的说明 |
| `entities` | [MessageEntity](#messageentity) 数组 | *可选。* 文本中的特殊实体（命令等） |
| `photo` | [PhotoSize](#photosize) 数组 | *可选。* 当消息为图片时存在 |
| `video` | [Video](#video) | *可选。* 当消息为视频时存在 |
| `document` | [Document](#document) | *可选。* 当消息为文档时存在 |
| `voice` | [Voice](#voice) | *可选。* 当消息为语音/音频时存在 |
| `location` | [Location](#location) | *可选。* 当消息为位置时存在 |
| `reply_to_message` | [Message](#message) | *可选。* 回复消息时，被引用的原始消息。仅展开一层：即使原始消息本身也是回复，也不会再包含 `reply_to_message` |
| `reply_markup` | [InlineKeyboardMarkup](#inlinekeyboardmarkup) | *可选。* 附加到消息的内联键盘 |

## PhotoSize

表示图片的一种尺寸。

| 字段 | 类型 | 说明 |
|------|------|------|
| `file_id` | String | 文件标识 / URL |
| `width` | Integer | *可选。* 图片宽度 |
| `height` | Integer | *可选。* 图片高度 |

## Video

| 字段 | 类型 | 说明 |
|------|------|------|
| `file_id` | String | 文件标识 / URL |
| `duration` | Integer | *可选。* 时长（秒） |
| `width` | Integer | *可选。* 视频宽度 |
| `height` | Integer | *可选。* 视频高度 |

## Document

| 字段 | 类型 | 说明 |
|------|------|------|
| `file_id` | String | 文件标识 / URL |
| `file_name` | String | *可选。* 原始文件名 |
| `file_size` | Integer | *可选。* 文件大小（字节） |

## Voice

| 字段 | 类型 | 说明 |
|------|------|------|
| `file_id` | String | 文件标识 / URL |
| `duration` | Integer | *可选。* 时长（秒） |

## Location

| 字段 | 类型 | 说明 |
|------|------|------|
| `latitude` | Float | 发送者定义的纬度 |
| `longitude` | Float | 发送者定义的经度 |

## MessageEntity

表示文本消息中的一个特殊实体（例如机器人命令）。

| 字段 | 类型 | 说明 |
|------|------|------|
| `type` | String | 实体类型。当前支持：`"bot_command"`、`"text_mention"`（群消息中的 `@` 提及） |
| `offset` | Integer | 实体起点的偏移量（以 UTF-16 码元计） |
| `length` | Integer | 实体长度（以 UTF-16 码元计） |
| `user` | [User](#user) | *可选。* 被提及的用户，`"text_mention"` 时存在。`first_name` 为该用户的群昵称 |

## CallbackQuery

表示由[内联键盘](#inlinekeyboardbutton)按钮按下产生的一个回调查询。

| 字段 | 类型 | 说明 |
|------|------|------|
| `id` | String | 该查询的唯一标识 |
| `from` | [User](#user) | 发送者 |
| `message` | [Message](#message) | *可选。* 触发该查询的带回调按钮的消息 |
| `data` | String | *可选。* 与回调按钮关联的数据（1-256 个字符） |

> **注意：** 收到回调查询后，机器人应调用 [answerCallbackQuery](METHODS.md#answercallbackquery) 进行确认。

## Update

表示一个传入的更新。以下可选字段中**最多**只会出现一个。

| 字段 | 类型 | 说明 |
|------|------|------|
| `update_id` | Integer | 更新的唯一标识，单调递增。 |
| `message` | [Message](#message) | *可选。* 新的传入消息 |
| `callback_query` | [CallbackQuery](#callbackquery) | *可选。* 新的传入回调查询 |

## InlineKeyboardMarkup

表示紧贴在消息下方显示的内联键盘。

| 字段 | 类型 | 说明 |
|------|------|------|
| `inline_keyboard` | [InlineKeyboardButton](#inlinekeyboardbutton) 的二维数组 | 按钮行数组，每一行是一个按钮数组 |

## InlineKeyboardButton

表示内联键盘上的一个按钮。`callback_data`、`url`、`web_app` 必须且只能指定其一。

| 字段 | 类型 | 说明 |
|------|------|------|
| `text` | String | 按钮上的文字 |
| `callback_data` | String | *可选。* 按钮被按下时在 [CallbackQuery](#callbackquery) 中发送的数据（1-256 个字符） |
| `url` | String | *可选。* 按钮被按下时打开的 HTTP/HTTPS URL |
| `web_app` | [WebAppInfo](#webappinfo) | *可选。* 按钮被按下时启动的 Web App |

用法示例与回调流程参见[内联键盘](KEYBOARDS.md)。

## WebAppInfo

描述一个 Web App（小程序）。NexLink 会在应用内的 dApp 浏览器中打开它并注入 `NexlinkApp` SDK。

| 字段 | 类型 | 说明 |
|------|------|------|
| `url` | String | 要打开的 Web App 的 HTTPS URL |

## BotCommand

表示一个机器人命令。

| 字段 | 类型 | 说明 |
|------|------|------|
| `command` | String | 命令文本（1-32 个字符）。仅小写 `a-z`、数字和下划线。不带前导 `/`。 |
| `description` | String | 命令说明（1-256 个字符） |

## WebhookInfo

包含当前 Webhook 状态的信息。

| 字段 | 类型 | 说明 |
|------|------|------|
| `url` | String | Webhook URL，未设置时为空字符串 |
| `has_custom_certificate` | Boolean | 始终为 `false`（保留字段，用于兼容） |
| `pending_update_count` | Integer | 等待投递的更新数量 |
| `last_error_message` | String | *可选。* 最近一次投递失败的错误信息 |
