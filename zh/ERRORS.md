# 错误与参考

- [错误处理](#错误处理)
- [消息流转](#消息流转)
- [OpenIM 内容类型映射](#openim-内容类型映射)
- [约束限制](#约束限制)

---

## 错误处理

### Bot API 错误

| HTTP 状态码 | 说明 |
|-------------|------|
| 200 | 成功 |
| 400 | 请求错误 —— 参数无效 |
| 401 | 未授权 —— 机器人令牌无效 |
| 404 | 未找到 —— 机器人、消息或查询不存在 |
| 409 | 冲突 —— 用户名已被占用 |
| 500 | 服务器内部错误 |

### 管理 API 错误

管理与发现 API 使用标准的 NexLink 错误信封：

```json
{
  "errCode": 1001,
  "errMsg": "bot not found",
  "errDlt": "the specified bot does not exist"
}
```

### 状态码

| 值 | 状态 | 说明 |
|-----|------|------|
| 1 | Active | 机器人正常运行 |
| 2 | Suspended | 机器人被临时停用 |
| 3 | Deleted | 机器人已被软删除 |

---

## 消息流转

```
┌─────────┐     ┌──────────────┐     ┌─────────────┐     ┌──────────┐
│  用户    │────▶│  OpenIM      │────▶│  NexLink    │────▶│  机器人   │
│ （App）  │     │  Server      │     │  API        │     │  Server  │
│          │◀────│              │◀────│  (webhook)  │◀────│          │
└─────────┘     └──────────────┘     └─────────────┘     └──────────┘

1. 用户在 NexLink App 中向机器人发送消息
2. OpenIM 投递消息并触发 afterSendSingleMsg 回调
3. NexLink API 拦截回调，构建 Update JSON
4. Webhook 分发器将 Update POST 到机器人的 Webhook URL
5. 机器人服务器处理 Update，调用 Bot API（sendMessage 等）
6. NexLink API 以机器人身份通过 OpenIM 发送消息
7. 用户在 NexLink App 中收到机器人的回复
```

---

## OpenIM 内容类型映射

| Bot API 方法 | OpenIM ContentType | 说明 |
|-------------|-------------------|------|
| sendMessage | 101 | 文本 |
| sendPhoto | 102 | 图片 |
| sendAudio | 103 | 声音 |
| sendVoice | 103 | 声音 |
| sendVideo | 104 | 视频 |
| sendDocument | 105 | 文件 |
| sendLocation | 114 | 位置 |

---

## 约束限制

| 项目 | 限制 |
|------|------|
| 机器人用户名 | 1-32 个字符，`[a-z0-9_]` |
| 命令名 | 1-32 个字符，`[a-z0-9_]`，无 `/` 前缀 |
| 命令说明 | 1-256 个字符 |
| 回调数据 | 1-256 个字符 |
| 消息文本 | 1-4096 个字符 |
| Webhook URL | 最多 512 个字符，HTTPS |
| Webhook 密钥 | 最多 256 个字符 |
| 每个机器人的命令数 | 最多 100 |
