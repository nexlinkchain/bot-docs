# Inline Keyboards

Inline keyboards are displayed below messages. Each button can open a URL, send a callback query to your bot, or launch a [Web App](#example-web-app-button). Attach one via the `reply_markup` field on [sendMessage](METHODS.md#sendmessage) and the other send methods. See the [InlineKeyboardMarkup](TYPES.md#inlinekeyboardmarkup) and [InlineKeyboardButton](TYPES.md#inlinekeyboardbutton) types.

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

## Example: Web App Button

A `web_app` button opens the URL inside NexLink's in-app dApp browser with the `NexlinkApp` SDK injected — the NexLink equivalent of a Telegram Web App (mini-app). No callback query is generated; the interaction continues inside the Web App.

```json
{
  "reply_markup": {
    "inline_keyboard": [
      [
        {"text": "Open Shop", "web_app": {"url": "https://shop.example.com"}}
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

After receiving a [CallbackQuery](TYPES.md#callbackquery), your bot should call [answerCallbackQuery](METHODS.md#answercallbackquery) to acknowledge it. The answer is pushed to the user in real time over the socket and shown as a toast (or an alert when `show_alert` is set); if the user has gone offline in the meantime it is simply not displayed.
