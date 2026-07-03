# NexLink Bot API

> HTTP API for building bots on NexLink.

Bots are special OpenIM accounts controlled programmatically. They can receive messages, send replies, present [inline keyboards](KEYBOARDS.md), and interact through [webhooks](WEBHOOKS.md).

---

## Getting Started

### 1. Create a bot

Use the [Bot Management API](MANAGEMENT.md#create-bot) or the NexLink app's **Bot Setup** screen to create a bot. You will receive a **token** — a string like this:

```
bot_a1b2c3d4:c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8
```

> **Keep your token secure.** Anyone with your token can control your bot. If it is compromised, [regenerate it](MANAGEMENT.md#regenerate-token) immediately.

### 2. Configure a webhook

Tell NexLink where to deliver updates by [setting a webhook](METHODS.md#setwebhook). NexLink will POST a JSON [Update](TYPES.md#update) object to your URL every time a user interacts with your bot.

### 3. Start receiving messages

When a user sends a message to your bot, NexLink delivers a webhook with a [Message](TYPES.md#message) object. Your server processes it and calls the Bot API to respond.

### Webhook or polling

There are two ways to receive updates, and you can use either:

- **Webhook** — NexLink pushes each [Update](TYPES.md#update) to your HTTPS URL. Best for production. See [setWebhook](METHODS.md#setwebhook).
- **Polling** — your bot pulls pending updates with [getUpdates](METHODS.md#getupdates). No public URL required, so it is ideal for local development and testing.

Pending updates are queued server-side for **every** bot, so `getUpdates` works whether or not a webhook is configured. If you enable a webhook *and* poll, each update is delivered through both paths — pick one to avoid processing every update twice.

---

## Making Requests

All methods are invoked via **HTTP POST** to:

```
https://<your-nexlink-api-host>/bot<token>/<method>
```

Example — sending a text message:

```bash
curl -X POST https://api.nexlink.im/botbot_a1b2c3d4:c9d0e1f2.../sendMessage \
  -H "Content-Type: application/json" \
  -d '{"chat_id": "user_abc123", "text": "Hello!"}'
```

### Response Format

Every response is a JSON object:

```json
{
  "ok": true,
  "result": { ... }
}
```

On error:

```json
{
  "ok": false,
  "error_code": 400,
  "description": "Bad Request: chat_id is required"
}
```

---

## Authorizing Your Bot

The bot token is embedded in the URL path. No additional headers are needed for Bot API calls.

```
/bot<token>/methodName
    ^^^^^^^
    Your secret token
```

The token format is `<bot_id>:<secret>`, e.g. `bot_a1b2c3d4:random40hex`. Only the SHA-256 hash is stored server-side.

---

## Documentation

| Chapter | Description |
|---|---|
| [Methods](METHODS.md) | All Bot API methods — sending, editing, polling, commands, webhook config |
| [Types](TYPES.md) | Object definitions used in requests, responses, and webhooks |
| [Inline Keyboards](KEYBOARDS.md) | Building inline keyboards and handling callback queries |
| [Webhooks](WEBHOOKS.md) | Webhook setup, payloads, signature verification, retry policy |
| [Management & Discovery](MANAGEMENT.md) | User-facing bot management and public discovery endpoints |
| [Errors & Reference](ERRORS.md) | Error codes, message flow, content-type mapping, constraints |
