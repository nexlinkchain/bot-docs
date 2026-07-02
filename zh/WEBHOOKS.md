# Webhooks

NexLink 可将每个 [Update](TYPES.md#update) 推送到你控制的一个 HTTPS URL。Webhook 是生产环境推荐的投递方式；本地开发时也可改用 [getUpdates](METHODS.md#getupdates) 轮询。

---

## 配置 Webhook

```bash
POST /bot<token>/setWebhook
Content-Type: application/json

{
  "url": "https://myserver.com/webhook/bot_a1b2c3d4",
  "secret_token": "my-verification-secret",
  "allowed_updates": ["message", "callback_query"]
}
```

要求：
- URL 必须为 HTTPS
- 你的服务器必须在 10 秒内返回 HTTP 200
- 非 200 响应会触发[重试逻辑](#重试与投递)

## Webhook 载荷

NexLink 将序列化为 JSON 的 [Update](TYPES.md#update) 作为 HTTP POST 请求体发送。

**消息更新：**

```json
{
  "update_id": 100001,
  "message": {
    "message_id": "srv_msg_a1b2c3d4",
    "from": {
      "id": "user_abc123",
      "is_bot": false,
      "first_name": "Alice",
      "username": "alice"
    },
    "chat": {
      "id": "user_abc123",
      "type": "private"
    },
    "date": 1719846000,
    "text": "/start",
    "entities": [
      {"type": "bot_command", "offset": 0, "length": 6}
    ]
  }
}
```

**回调查询更新：**

```json
{
  "update_id": 100002,
  "callback_query": {
    "id": "cbq_e5f6a7b8",
    "from": {
      "id": "user_abc123",
      "is_bot": false,
      "first_name": "Alice"
    },
    "message": {
      "message_id": "srv_msg_a1b2c3d4",
      "chat": {
        "id": "user_abc123",
        "type": "private"
      }
    },
    "data": "delete_confirm"
  }
}
```

## Webhook 请求头

每个 Webhook 请求都包含以下请求头：

| 请求头 | 说明 |
|--------|------|
| `Content-Type` | `application/json` |
| `X-NexLink-Bot-Secret-Token` | 你在 [setWebhook](METHODS.md#setwebhook) 中提供的 `secret_token`。用于验证请求确实来自 NexLink。 |
| `X-Nexlink-Timestamp` | 请求的 Unix 时间戳 |
| `X-Nexlink-Signature` | 请求体的 HMAC-SHA256 十六进制摘要，使用 Webhook 密钥签名 |

**验证示例（Node.js）：**

```javascript
const crypto = require('crypto');

function verifyWebhook(req, secret) {
  const signature = req.headers['x-nexlink-signature'];
  const expected = crypto
    .createHmac('sha256', secret)
    .update(req.body)
    .digest('hex');
  return crypto.timingSafeEqual(
    Buffer.from(signature, 'hex'),
    Buffer.from(expected, 'hex')
  );
}
```

**验证示例（Python）：**

```python
import hmac, hashlib

def verify_webhook(body: bytes, secret: str, signature: str) -> bool:
    expected = hmac.new(
        secret.encode(), body, hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(expected, signature)
```

**验证示例（Go）：**

```go
func verifyWebhook(body []byte, secret, signature string) bool {
    mac := hmac.New(sha256.New, []byte(secret))
    mac.Write(body)
    expected := hex.EncodeToString(mac.Sum(nil))
    return hmac.Equal([]byte(expected), []byte(signature))
}
```

## 重试与投递

| 项目 | 值 |
|------|-----|
| 工作协程池 | 4 个并发 worker |
| 单次投递超时 | 10 秒 |
| 最大尝试次数 | 8 |
| 退避策略 | 指数退避：30s → 1m → 2m → 4m → 8m → 16m → 30m（上限） |
| 清扫器 | 后台每 30 秒清扫一次滞留的待投递记录 |

8 次尝试失败后，该投递被标记为**失败**。机器人记录上的 `last_error` 和 `last_error_at` 字段会被更新。
