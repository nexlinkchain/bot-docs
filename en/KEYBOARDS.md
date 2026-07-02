# Inline Keyboards

Inline keyboards are displayed below messages. Each button can either open a URL or send a callback query to your bot. Attach one via the `reply_markup` field on [sendMessage](METHODS.md#sendmessage) and the other send methods. See the [InlineKeyboardMarkup](TYPES.md#inlinekeyboardmarkup) and [InlineKeyboardButton](TYPES.md#inlinekeyboardbutton) types.

---

## Example: Confirmation Dialog

```json
{
  "chat_id": "user_abc123",
  "text": "Are you sure you want to delete this item?",
  "reply_markup": {
    "inline_keyboard": [
      [
        {"text": "Yes, delete", "callback_data": "delete_confirm"},
        {"text": "Cancel", "callback_data": "delete_cancel"}
      ]
    ]
  }
}
```

## Example: Multi-Row with URL

```json
{
  "reply_markup": {
    "inline_keyboard": [
      [
        {"text": "Option A", "callback_data": "opt_a"},
        {"text": "Option B", "callback_data": "opt_b"}
      ],
      [
        {"text": "Visit Website", "url": "https://nexlink.im"}
      ]
    ]
  }
}
```

## How Callback Buttons Work

```
User taps button
       │
       ▼
NexLink app sends POST /bot_mgmt/callback_query
  { botId, chatId, messageId, data: "delete_confirm" }
       │
       ▼
NexLink server creates CallbackQuery record
       │
       ▼
Webhook delivered to bot backend
  { update_id: N, callback_query: { id, from, message, data } }
       │
       ▼
Bot calls POST /bot<token>/answerCallbackQuery
  { callback_query_id: "...", text: "Item deleted!" }
```

After receiving a [CallbackQuery](TYPES.md#callbackquery), your bot should call [answerCallbackQuery](METHODS.md#answercallbackquery) to acknowledge it.
