# Types

Object definitions used across the Bot API — in method parameters, responses, and [webhook](WEBHOOKS.md) payloads.

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

Represents a NexLink user or bot.

| Field | Type | Description |
|-------|------|-------------|
| `id` | String | Unique identifier (OpenIM userID) |
| `is_bot` | Boolean | `true` if this user is a bot |
| `first_name` | String | User's or bot's display name |
| `username` | String | *Optional.* User's or bot's username |

## Chat

Represents a chat.

| Field | Type | Description |
|-------|------|-------------|
| `id` | String | Unique identifier for this chat (OpenIM userID or groupID) |
| `type` | String | Type of chat: `"private"`, `"group"`, or `"channel"` |

## Message

Represents a message.

| Field | Type | Description |
|-------|------|-------------|
| `message_id` | String | Unique message identifier (OpenIM serverMsgID) |
| `from` | [User](#user) | *Optional.* Sender of the message |
| `chat` | [Chat](#chat) | Conversation the message belongs to |
| `date` | Integer | Date the message was sent (Unix timestamp) |
| `text` | String | *Optional.* Text content of the message |
| `caption` | String | *Optional.* Caption for media messages |
| `entities` | Array of [MessageEntity](#messageentity) | *Optional.* Special entities in the text (commands, etc.) |
| `photo` | Array of [PhotoSize](#photosize) | *Optional.* Available if the message is a photo |
| `video` | [Video](#video) | *Optional.* Available if the message is a video |
| `document` | [Document](#document) | *Optional.* Available if the message is a document |
| `voice` | [Voice](#voice) | *Optional.* Available if the message is a voice/audio message |
| `location` | [Location](#location) | *Optional.* Available if the message is a location |
| `reply_to_message` | [Message](#message) | *Optional.* For replies, the original message being quoted. Exposed one level deep: it will not contain a further `reply_to_message` even if it is itself a reply |
| `reply_markup` | [InlineKeyboardMarkup](#inlinekeyboardmarkup) | *Optional.* Inline keyboard attached to the message |

## PhotoSize

Represents one size of a photo.

| Field | Type | Description |
|-------|------|-------------|
| `file_id` | String | Identifier / URL for the file |
| `width` | Integer | *Optional.* Photo width |
| `height` | Integer | *Optional.* Photo height |

## Video

| Field | Type | Description |
|-------|------|-------------|
| `file_id` | String | Identifier / URL for the file |
| `duration` | Integer | *Optional.* Duration in seconds |
| `width` | Integer | *Optional.* Video width |
| `height` | Integer | *Optional.* Video height |

## Document

| Field | Type | Description |
|-------|------|-------------|
| `file_id` | String | Identifier / URL for the file |
| `file_name` | String | *Optional.* Original filename |
| `file_size` | Integer | *Optional.* File size in bytes |

## Voice

| Field | Type | Description |
|-------|------|-------------|
| `file_id` | String | Identifier / URL for the file |
| `duration` | Integer | *Optional.* Duration in seconds |

## Location

| Field | Type | Description |
|-------|------|-------------|
| `latitude` | Float | Latitude as defined by the sender |
| `longitude` | Float | Longitude as defined by the sender |

## MessageEntity

Represents a special entity in a text message (e.g., a bot command).

| Field | Type | Description |
|-------|------|-------------|
| `type` | String | Type of the entity. Currently supported: `"bot_command"`, `"text_mention"` (an `@` mention in a group message) |
| `offset` | Integer | Offset in UTF-16 code units to the start of the entity |
| `length` | Integer | Length of the entity in UTF-16 code units |
| `user` | [User](#user) | *Optional.* The mentioned user, present for `"text_mention"`. `first_name` carries the user's group nickname |

## CallbackQuery

Represents an incoming callback query from a button press on an [inline keyboard](#inlinekeyboardbutton).

| Field | Type | Description |
|-------|------|-------------|
| `id` | String | Unique identifier for this query |
| `from` | [User](#user) | Sender |
| `message` | [Message](#message) | *Optional.* Message with the callback button that originated the query |
| `data` | String | *Optional.* Data associated with the callback button (1-256 characters) |

> **Note:** After receiving a callback query, your bot should call [answerCallbackQuery](METHODS.md#answercallbackquery) to acknowledge it.

## Update

Represents an incoming update. At most **one** of the optional fields will be present.

| Field | Type | Description |
|-------|------|-------------|
| `update_id` | Integer | Unique update identifier. Monotonically increasing. |
| `message` | [Message](#message) | *Optional.* New incoming message |
| `callback_query` | [CallbackQuery](#callbackquery) | *Optional.* New incoming callback query |

## InlineKeyboardMarkup

Represents an inline keyboard that appears right below the message.

| Field | Type | Description |
|-------|------|-------------|
| `inline_keyboard` | Array of Array of [InlineKeyboardButton](#inlinekeyboardbutton) | Array of button rows, each row is an array of buttons |

## InlineKeyboardButton

Represents one button of an inline keyboard. Exactly one of `callback_data`, `url`, or `web_app` must be specified.

| Field | Type | Description |
|-------|------|-------------|
| `text` | String | Label text on the button |
| `callback_data` | String | *Optional.* Data sent in a [CallbackQuery](#callbackquery) when the button is pressed (1-256 characters) |
| `url` | String | *Optional.* HTTP/HTTPS URL to be opened when button is pressed |
| `web_app` | [WebAppInfo](#webappinfo) | *Optional.* Web App launched when the button is pressed |

See [Inline Keyboards](KEYBOARDS.md) for usage examples and the callback flow.

## WebAppInfo

Describes a Web App (mini-app). NexLink opens it inside the app's dApp browser with the `NexlinkApp` SDK injected.

| Field | Type | Description |
|-------|------|-------------|
| `url` | String | HTTPS URL of the Web App to open |

## BotCommand

Represents a bot command.

| Field | Type | Description |
|-------|------|-------------|
| `command` | String | Command text (1-32 characters). Lowercase `a-z`, digits, and underscores only. No leading `/`. |
| `description` | String | Description of the command (1-256 characters) |

## WebhookInfo

Contains information about the current status of a webhook.

| Field | Type | Description |
|-------|------|-------------|
| `url` | String | Webhook URL, empty string if webhook is not set up |
| `has_custom_certificate` | Boolean | Always `false` (reserved for compatibility) |
| `pending_update_count` | Integer | Number of updates awaiting delivery |
| `last_error_message` | String | *Optional.* Error message from the most recent failed delivery attempt |
