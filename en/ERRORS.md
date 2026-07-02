# Errors & Reference

- [Error Handling](#error-handling)
- [Message Flow](#message-flow)
- [OpenIM Content Type Mapping](#openim-content-type-mapping)
- [Constraints](#constraints)

---

## Error Handling

### Bot API Errors

| HTTP Code | Description |
|-----------|-------------|
| 200 | Success |
| 400 | Bad Request — invalid parameters |
| 401 | Unauthorized — invalid bot token |
| 404 | Not Found — bot, message, or query not found |
| 409 | Conflict — username already taken |
| 500 | Internal Server Error |

### Management API Errors

Management and discovery APIs use the standard NexLink error envelope:

```json
{
  "errCode": 1001,
  "errMsg": "bot not found",
  "errDlt": "the specified bot does not exist"
}
```

### Status Codes

| Value | Status | Description |
|-------|--------|-------------|
| 1 | Active | Bot is operational |
| 2 | Suspended | Bot is temporarily suspended |
| 3 | Deleted | Bot has been soft-deleted |

---

## Message Flow

```
┌─────────┐     ┌──────────────┐     ┌─────────────┐     ┌──────────┐
│  User    │────▶│  OpenIM      │────▶│  NexLink    │────▶│  Bot     │
│  (app)   │     │  Server      │     │  API        │     │  Server  │
│          │◀────│              │◀────│  (webhook)  │◀────│          │
└─────────┘     └──────────────┘     └─────────────┘     └──────────┘

1. User sends message to bot in NexLink app
2. OpenIM delivers message, fires afterSendSingleMsg callback
3. NexLink API intercepts callback, builds Update JSON
4. Webhook dispatcher POSTs Update to bot's webhook URL
5. Bot server processes Update, calls Bot API (sendMessage, etc.)
6. NexLink API sends message via OpenIM as the bot user
7. User receives bot's reply in NexLink app
```

---

## OpenIM Content Type Mapping

| Bot API Method | OpenIM ContentType | Description |
|---------------|-------------------|-------------|
| sendMessage | 101 | Text |
| sendPhoto | 102 | Picture |
| sendAudio | 103 | Sound |
| sendVoice | 103 | Sound |
| sendVideo | 104 | Video |
| sendDocument | 105 | File |
| sendLocation | 114 | Location |

---

## Constraints

| Item | Limit |
|------|-------|
| Bot username | 1-32 characters, `[a-z0-9_]` |
| Command name | 1-32 characters, `[a-z0-9_]`, no `/` prefix |
| Command description | 1-256 characters |
| Callback data | 1-256 characters |
| Message text | 1-4096 characters |
| Webhook URL | Max 512 characters, HTTPS |
| Webhook secret | Max 256 characters |
| Commands per bot | Max 100 |
