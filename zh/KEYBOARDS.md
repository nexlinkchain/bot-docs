# 内联键盘

内联键盘显示在消息下方。每个按钮可以打开一个 URL、向你的机器人发送回调查询，或启动一个 [Web App](#示例web-app-按钮)。通过 [sendMessage](METHODS.md#sendmessage) 及其他发送方法的 `reply_markup` 字段附加内联键盘。参见 [InlineKeyboardMarkup](TYPES.md#inlinekeyboardmarkup) 与 [InlineKeyboardButton](TYPES.md#inlinekeyboardbutton) 类型。

---

## 示例：确认对话框

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

## 示例：多行 + URL

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

## 示例：Web App 按钮

`web_app` 按钮会在 NexLink 应用内的 dApp 浏览器中打开该 URL 并注入 `NexlinkApp` SDK——相当于 Telegram 的 Web App（小程序）。不会产生回调查询；后续交互在 Web App 内部完成。

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

## 回调按钮的工作原理

```
用户点击按钮
       │
       ▼
NexLink App 发送 POST /bot_mgmt/callback_query
  { botId, chatId, messageId, data: "delete_confirm" }
       │
       ▼
NexLink 服务端创建 CallbackQuery 记录
       │
       ▼
通过 Webhook 投递到机器人后端
  { update_id: N, callback_query: { id, from, message, data } }
       │
       ▼
机器人调用 POST /bot<token>/answerCallbackQuery
  { callback_query_id: "...", text: "Item deleted!" }
```

收到 [CallbackQuery](TYPES.md#callbackquery) 后，机器人应调用 [answerCallbackQuery](METHODS.md#answercallbackquery) 进行确认。答案会通过 socket 实时推送给用户，显示为顶部提示（设置 `show_alert` 时显示为弹窗）；若用户此时已离线则不再显示。
