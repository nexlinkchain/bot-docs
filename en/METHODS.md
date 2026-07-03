# Methods

All methods are invoked via **HTTP POST** to `https://<your-nexlink-api-host>/bot<token>/<method>`. See [Making Requests](README.md#making-requests) for the request/response format.

- [getMe](#getme)
- [getUpdates](#getupdates)
- [sendMessage](#sendmessage)
- [sendPhoto](#sendphoto)
- [sendVideo](#sendvideo)
- [sendDocument](#senddocument)
- [sendAudio](#sendaudio)
- [sendVoice](#sendvoice)
- [sendLocation](#sendlocation)
- [sendChatAction](#sendchataction)
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

Returns basic information about the bot.

**Parameters:** None

**Returns:** [User](TYPES.md#user)

```bash
POST /bot<token>/getMe
```

**Response:**

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

Long-polling alternative to [webhooks](WEBHOOKS.md). Returns pending [Update](TYPES.md#update) objects for the bot. Updates are stored server-side for every bot regardless of webhook configuration, so you can poll even without setting a webhook.

**Parameters:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `offset` | Integer | No | Return updates with `update_id >= offset`. Passing an offset also **acknowledges** (deletes) every update with a lower `update_id`, so they are not returned again. Set this to `last_update_id + 1`. |
| `limit` | Integer | No | Maximum number of updates to return, 1-100. Defaults to 100. |
| `timeout` | Integer | No | Long-polling timeout in seconds, 0-50. `0` returns immediately (short poll). If no updates are pending, the request waits up to `timeout` seconds for one to arrive. Defaults to 0. |

**Returns:** Array of [Update](TYPES.md#update). Returns an empty array if none arrive before the timeout.

```bash
POST /bot<token>/getUpdates
Content-Type: application/json

{
  "offset": 100002,
  "limit": 100,
  "timeout": 30
}
```

**Response:**

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

> **Polling loop:** call `getUpdates` in a loop. On each response, remember the highest `update_id`, then pass `offset = highest + 1` on the next call to acknowledge the batch and fetch only newer updates.

---

## sendMessage

Sends a text message.

**Parameters:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `chat_id` | String | Yes | Unique identifier for the target chat (OpenIM userID or groupID) |
| `text` | String | Yes | Text of the message, 1-4096 characters |
| `reply_markup` | [InlineKeyboardMarkup](TYPES.md#inlinekeyboardmarkup) | No | Inline keyboard attached to the message |

**Returns:** [Message](TYPES.md#message)

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

> **Shared media fields.** In addition to the parameters listed per method below, every media send method (`sendPhoto`, `sendVideo`, `sendDocument`, `sendAudio`, `sendVoice`) accepts these optional fields. They are passed through to the message so clients can render richer previews:
>
> | Field | Type | Description |
> |-------|------|-------------|
> | `mime_type` | String | MIME type of the file |
> | `file_size` | Integer | File size in bytes |
> | `file_name` | String | Original filename (documents) |
> | `width` | Integer | Media width in pixels (photo/video) |
> | `height` | Integer | Media height in pixels (photo/video) |
> | `duration` | Integer | Duration in seconds (audio/voice/video) |
> | `thumb_url` | String | Thumbnail URL (video) |
> | `thumb_width` | Integer | Thumbnail width in pixels |
> | `thumb_height` | Integer | Thumbnail height in pixels |

## sendPhoto

Sends a photo.

**Parameters:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `chat_id` | String | Yes | Target chat ID |
| `url` | String | Yes | URL of the photo |
| `caption` | String | No | Photo caption, 0-1024 characters |
| `reply_markup` | [InlineKeyboardMarkup](TYPES.md#inlinekeyboardmarkup) | No | Inline keyboard |

**Returns:** [Message](TYPES.md#message)

---

## sendVideo

Sends a video.

**Parameters:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `chat_id` | String | Yes | Target chat ID |
| `url` | String | Yes | URL of the video |
| `caption` | String | No | Video caption |
| `reply_markup` | [InlineKeyboardMarkup](TYPES.md#inlinekeyboardmarkup) | No | Inline keyboard |

**Returns:** [Message](TYPES.md#message)

---

## sendDocument

Sends a general file (document).

**Parameters:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `chat_id` | String | Yes | Target chat ID |
| `url` | String | Yes | URL of the document |
| `caption` | String | No | Document caption |
| `reply_markup` | [InlineKeyboardMarkup](TYPES.md#inlinekeyboardmarkup) | No | Inline keyboard |

**Returns:** [Message](TYPES.md#message)

---

## sendAudio

Sends an audio file.

**Parameters:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `chat_id` | String | Yes | Target chat ID |
| `url` | String | Yes | URL of the audio |
| `caption` | String | No | Audio caption |
| `reply_markup` | [InlineKeyboardMarkup](TYPES.md#inlinekeyboardmarkup) | No | Inline keyboard |

**Returns:** [Message](TYPES.md#message)

---

## sendVoice

Sends a voice message.

**Parameters:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `chat_id` | String | Yes | Target chat ID |
| `url` | String | Yes | URL of the voice recording |
| `caption` | String | No | Voice caption |
| `reply_markup` | [InlineKeyboardMarkup](TYPES.md#inlinekeyboardmarkup) | No | Inline keyboard |

**Returns:** [Message](TYPES.md#message)

---

## sendLocation

Sends a point on the map.

**Parameters:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `chat_id` | String | Yes | Target chat ID |
| `latitude` | Float | Yes | Latitude of the location |
| `longitude` | Float | Yes | Longitude of the location |
| `reply_markup` | [InlineKeyboardMarkup](TYPES.md#inlinekeyboardmarkup) | No | Inline keyboard |

**Returns:** [Message](TYPES.md#message)

---

## sendChatAction

Tells the user that something is happening on the bot's side — the client shows a "Typing..." status in the chat header. Use while a request takes a noticeable time to process, right before the actual reply.

The status is delivered online-only (nothing is stored) and clears automatically after ~6 seconds or as soon as the bot sends a message — whichever comes first. Refresh it by calling the method again for longer operations.

**Parameters:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `chat_id` | String | Yes | Target chat ID (a user's ID) |
| `action` | String | Yes | Type of action: `typing`, `upload_photo`, `record_video`, `upload_video`, `record_voice`, `upload_voice`, `upload_document`, `choose_sticker`, `find_location`. All actions currently render as "Typing..." |

**Returns:** `true` on success.

```bash
POST /bot<token>/sendChatAction
Content-Type: application/json

{
  "chat_id": "5168018091",
  "action": "typing"
}
```

---

## forwardMessage

Copies the content of a message the bot previously sent or received into another chat. Uses copy semantics (no "forwarded from" header), mirroring Telegram's behavior. Only messages the bot has a record of can be forwarded.

**Parameters:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `chat_id` | String | Yes | Target chat to copy the message into |
| `message_id` | String | Yes | `message_id` of a message known to the bot (one it sent or received) |
| `from_chat_id` | String | No | Source chat id (reserved; the message is located by `message_id`) |

**Returns:** [Message](TYPES.md#message)

> Returns `400 Bad Request: message to forward not found` if the `message_id` is not known to the bot.

---

## editMessageText

Edits the text of a message the bot sent. OpenIM has no in-place edit, so the original is revoked and a replacement is sent — **the returned message has a new `message_id`**, which you should store as the current message.

**Parameters:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `chat_id` | String | No | Target chat (informational) |
| `message_id` | String | Yes | `message_id` of the bot's own message to edit |
| `text` | String | Yes | New text, 1-4096 characters |
| `reply_markup` | [InlineKeyboardMarkup](TYPES.md#inlinekeyboardmarkup) | No | New inline keyboard |

**Returns:** [Message](TYPES.md#message) — with a **new** `message_id`.

> Errors: `400 Bad Request: message not found`, or `400 Bad Request: can only edit the bot's own messages`.

---

## deleteMessage

Deletes (revokes) a message the bot sent.

**Parameters:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `chat_id` | String | No | Target chat (informational) |
| `message_id` | String | Yes | `message_id` of a message known to the bot |

**Returns:** `true` on success.

> Returns `400 Bad Request: message not found` if the `message_id` is not known to the bot.

---

## answerCallbackQuery

Sends an answer to a callback query sent from an [inline keyboard](TYPES.md#inlinekeyboardbutton). The answer is pushed to the user in real time and displayed as a notification at the top of the chat screen or as an alert. Delivery is online-only: if the user has gone offline, the answer is not shown later.

**Parameters:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `callback_query_id` | String | Yes | Unique identifier for the query to be answered |
| `text` | String | No | Text of the notification (0-200 characters) |
| `show_alert` | Boolean | No | If `true`, show an alert instead of a notification at the top of the chat screen. Defaults to `false` |

**Returns:** `true` on success.

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

Sets the list of the bot's commands. Users see these when they type `/` in the chat.

**Parameters:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `commands` | Array of [BotCommand](TYPES.md#botcommand) | Yes | List of bot commands to be set. At most 100 commands. |

**Returns:** `true` on success.

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

Gets the current list of the bot's commands.

**Parameters:** None

**Returns:** Array of [BotCommand](TYPES.md#botcommand)

---

## deleteMyCommands

Deletes the list of the bot's commands.

**Parameters:** None

**Returns:** `true` on success.

---

## setWebhook

Specifies a URL that NexLink will POST [Update](TYPES.md#update) objects to.

**Parameters:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `url` | String | Yes | HTTPS URL to send updates to |
| `secret_token` | String | No | A secret token (1-256 characters) sent in the `X-NexLink-Bot-Secret-Token` header for every webhook request |
| `allowed_updates` | Array of String | No | List of update types to receive. E.g. `["message", "callback_query"]`. Receive all by default. |

**Returns:** `true` on success.

```bash
POST /bot<token>/setWebhook
Content-Type: application/json

{
  "url": "https://myserver.com/bot/webhook",
  "secret_token": "mysecret123",
  "allowed_updates": ["message", "callback_query"]
}
```

See [Webhooks](WEBHOOKS.md) for payload format, signature verification, and retry policy.

---

## deleteWebhook

Removes webhook integration.

**Parameters:** None

**Returns:** `true` on success.

---

## getWebhookInfo

Returns current webhook status.

**Parameters:** None

**Returns:** [WebhookInfo](TYPES.md#webhookinfo)
