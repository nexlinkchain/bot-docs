# Management & Discovery

User-facing APIs for creating, configuring, and finding bots. These are distinct from the [Bot API methods](METHODS.md) (`/bot<token>/*`) — they are called by the NexLink app on behalf of a signed-in user, not by the bot itself.

- [Bot Management API](#bot-management-api)
- [Bot Discovery API](#bot-discovery-api)

---

## Bot Management API

User-facing API for creating and managing bots. All endpoints use `POST` and require the `CheckNexlinkToken` auth header (NexLink user session).

**Base path:** `/bot_mgmt`

### Create Bot

Creates a new bot. Returns the bot info and a **token** (shown only once).

```
POST /bot_mgmt/create
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `username` | String | Yes | Unique handle, 1-32 characters (a-z, 0-9, underscore) |
| `displayName` | String | Yes | Display name |
| `about` | String | No | Short description (max 512 characters) |
| `avatarUrl` | String | No | Avatar image URL |

**Response:**

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

> **Important:** The `token` is only returned at creation time. Store it securely.

---

### Update Bot

Updates bot metadata. Only the bot owner can update.

```
POST /bot_mgmt/update
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `botId` | String | Yes | Bot ID |
| `displayName` | String | No | New display name |
| `about` | String | No | New about text |
| `description` | String | No | Longer description (max 2048 characters) |
| `avatarUrl` | String | No | New avatar URL |

---

### Delete Bot

Soft-deletes a bot. Only the bot owner can delete.

```
POST /bot_mgmt/delete
```

| Field | Type | Required |
|-------|------|----------|
| `botId` | String | Yes |

---

### List My Bots

Lists all bots owned by the authenticated user.

```
POST /bot_mgmt/my_bots
```

**Request:** `{}`

**Response:** Array of bot objects.

---

### Regenerate Token

Invalidates the current token and returns a new one.

```
POST /bot_mgmt/regenerate_token
```

| Field | Type | Required |
|-------|------|----------|
| `botId` | String | Yes |

**Response:**

```json
{
  "errCode": 0,
  "data": {
    "token": "bot_a1b2c3d4:new_random_40_hex_characters_here_1234"
  }
}
```

---

### Set Webhook (Management)

Configures webhook delivery from the NexLink app UI.

```
POST /bot_mgmt/set_webhook
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `botId` | String | Yes | Bot ID |
| `url` | String | Yes | Webhook URL (HTTPS) |
| `secret` | String | No | Webhook secret for signature verification |
| `allowedUpdates` | Array of String | No | `["message", "callback_query"]` |

---

### Delete Webhook (Management)

Disables webhook delivery.

```
POST /bot_mgmt/delete_webhook
```

| Field | Type | Required |
|-------|------|----------|
| `botId` | String | Yes |

---

### Get Webhook Info (Management)

Returns webhook configuration and status.

```
POST /bot_mgmt/get_webhook_info
```

| Field | Type | Required |
|-------|------|----------|
| `botId` | String | Yes |

**Response:**

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

Makes the bot visible in the public bot store.

```
POST /bot_mgmt/publish
```

| Field | Type | Required |
|-------|------|----------|
| `botId` | String | Yes |

---

### Unpublish Bot

Removes the bot from the public bot store. Reverses [Publish Bot](#publish-bot).

```
POST /bot_mgmt/unpublish
```

| Field | Type | Required |
|-------|------|----------|
| `botId` | String | Yes |

---

### Submit Callback Query

Called by the NexLink Flutter client when a user taps an inline keyboard button. Creates a callback query record and enqueues it for webhook delivery.

```
POST /bot_mgmt/callback_query
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `botId` | String | Yes | Bot that sent the keyboard |
| `chatId` | String | Yes | OpenIM userID or groupID |
| `messageId` | String | Yes | OpenIM serverMsgID of the message with the keyboard |
| `data` | String | Yes | `callback_data` value from the button |

**Response:**

```json
{
  "errCode": 0,
  "data": {
    "queryId": "cbq_e5f6a7b8"
  }
}
```

---

## Bot Discovery API

Public endpoints for finding bots. All use `POST` with `CheckNexlinkToken` auth.

**Base path:** `/bots`

### Search Bots

Search published bots by keyword.

```
POST /bots/search
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `keyword` | String | No | Search query |
| `offset` | Integer | No | Pagination offset (default: 0) |
| `limit` | Integer | No | Results per page (default: 20, max: 100) |

---

### Featured Bots

Returns featured/recommended published bots.

```
POST /bots/featured
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `limit` | Integer | No | Max results (default: 20, max: 50) |

---

### Bot Profile

Gets a bot's public profile by ID or username.

```
POST /bots/profile
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `botId` | String | No | Bot ID (one of `botId` or `username` required) |
| `username` | String | No | Bot username (one of `botId` or `username` required) |

---

### Bot Commands

Gets a bot's public command list.

```
POST /bots/commands
```

| Field | Type | Required |
|-------|------|----------|
| `botId` | String | Yes |

**Response:**

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

## Deep Linking

Every bot is addressable by URL — NexLink's equivalent of Telegram's `t.me/bot?start=` links:

```
https://{web-domain}/bot/{bot_id}?start={payload}
nexlink://bot?id={bot_id}&start={payload}
```

Opening either link takes the user straight to the bot's chat. When a `start` payload is present, the client shows a **Start** button (even if the chat already has history); pressing it sends `/start {payload}` — so the payload is only delivered with the user's consent, exactly like Telegram.

The bot receives it as a plain text message:

```json
{"message": {"text": "/start ref_711", "from": {"id": "5168018091"}, ...}}
```

Use the payload for referral tracking, account linking (put a one-time token in it), or jumping to specific content. The payload should be URL-safe; keep it under 64 characters.
