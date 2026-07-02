# Webhooks

NexLink can push each [Update](TYPES.md#update) to an HTTPS URL you control. Webhooks are the recommended delivery model for production; for local development you can use [getUpdates](METHODS.md#getupdates) polling instead.

---

## Setting Up a Webhook

```bash
POST /bot<token>/setWebhook
Content-Type: application/json

{
  "url": "https://myserver.com/webhook/bot_a1b2c3d4",
  "secret_token": "my-verification-secret",
  "allowed_updates": ["message", "callback_query"]
}
```

Requirements:
- URL must be HTTPS
- Your server must respond with HTTP 200 within 10 seconds
- Non-200 responses trigger [retry logic](#retry--delivery)

## Webhook Payload

NexLink sends a JSON-serialized [Update](TYPES.md#update) as the body of an HTTP POST request.

**Message update:**

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

**Callback query update:**

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

## Webhook Headers

Every webhook request includes these headers:

| Header | Description |
|--------|-------------|
| `Content-Type` | `application/json` |
| `X-NexLink-Bot-Secret-Token` | The `secret_token` you provided in [setWebhook](METHODS.md#setwebhook). Use this to verify the request is from NexLink. |
| `X-Nexlink-Timestamp` | Unix timestamp of the request |
| `X-Nexlink-Signature` | HMAC-SHA256 hex digest of the request body, signed with the webhook secret |

**Verification example (Node.js):**

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

**Verification example (Python):**

```python
import hmac, hashlib

def verify_webhook(body: bytes, secret: str, signature: str) -> bool:
    expected = hmac.new(
        secret.encode(), body, hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(expected, signature)
```

**Verification example (Go):**

```go
func verifyWebhook(body []byte, secret, signature string) bool {
    mac := hmac.New(sha256.New, []byte(secret))
    mac.Write(body)
    expected := hex.EncodeToString(mac.Sum(nil))
    return hmac.Equal([]byte(expected), []byte(signature))
}
```

## Retry & Delivery

| Aspect | Value |
|--------|-------|
| Worker pool | 4 concurrent workers |
| Timeout | 10 seconds per delivery |
| Max attempts | 8 |
| Backoff | Exponential: 30s → 1m → 2m → 4m → 8m → 16m → 30m (cap) |
| Sweeper | Background sweep every 30 seconds for stale pending deliveries |

After 8 failed attempts, the delivery is marked as **failed**. The `last_error` and `last_error_at` fields on the bot record are updated.
