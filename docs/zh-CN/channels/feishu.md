---
summary: "飞书机器人概览、功能和配置"
read_when:
  - 你想连接飞书/Lark 机器人时
  - 你在配置飞书通道时
title: 飞书
---

飞书/Lark 是一个一站式协作平台，团队可以在这里聊天、分享文档、管理日历和共同完成工作。

**状态：** 私聊 DM + 群聊已具备生产就绪。WebSocket 是默认模式；webhook 模式是可选的。

---

## 快速开始

<Note>
需要 OpenClaw 2026.4.25 或更高版本。运行 `openclaw --version` 检查。使用 `openclaw update` 升级。
</Note>

<Steps>
  <Step title="运行通道设置向导">
  ```bash
  openclaw channels login --channel feishu
  ```
  选择手动设置以粘贴来自飞书开放平台的 App ID 和 App Secret，或选择二维码设置以自动创建机器人。如果国内飞书手机应用对二维码无反应，重新运行设置并选择手动设置。
  </Step>
  
  <Step title="设置完成后，重启 Gateway 以应用更改">
  ```bash
  openclaw gateway restart
  ```
  </Step>
</Steps>

---

## 访问控制

### 私聊消息

配置 `dmPolicy` 来控制谁可以给机器人发私聊：

- `"pairing"` - 未知用户会收到配对码；通过 CLI 批准
- `"allowlist"` - 仅 `allowFrom` 中列出的用户可以聊天（默认：仅机器人所有者）
- `"open"` - 仅当 `allowFrom` 包含 `"*"` 时允许公开 DM；有限制条目时，仅匹配的用户可以聊天
- `"disabled"` - 禁用所有 DM

**批准配对请求：**

```bash
openclaw pairing list feishu
openclaw pairing approve feishu <CODE>
```

### 群聊

**群策略**（`channels.feishu.groupPolicy`）：

| 值 | 行为 |
| --- | --- |
| `"open"` | 响应群内所有消息 |
| `"allowlist"` | 仅响应 `groupAllowFrom` 中列出的群或 `groups.<chat_id>` 下显式配置的群 |
| `"disabled"` | 禁用所有群消息；显式的 `groups.<chat_id>` 条目不会覆盖此设置 |

默认：`allowlist`

**提及要求**（`channels.feishu.requireMention`）：

- `true` - 需要 @提及（默认）
- `false` - 无需 @提及即可响应
- 每个群的覆盖设置：`channels.feishu.groups.<chat_id>.requireMention`
- 仅广播的 `@all` 和 `@_all` 不被视为机器人提及。同时提及了 `@all` 和直接提及机器人的消息仍计为机器人提及。

---

## 群配置示例

### 允许所有群，无需 @提及

```json5
{
  channels: {
    feishu: {
      groupPolicy: "open",
    },
  },
}
```

### 允许所有群，仍需 @提及

```json5
{
  channels: {
    feishu: {
      groupPolicy: "open",
      requireMention: true,
    },
  },
}
```

### 仅允许特定的群

```json5
{
  channels: {
    feishu: {
      groupPolicy: "allowlist",
      // 群 ID 格式如：oc_xxx
      groupAllowFrom: ["oc_xxx", "oc_yyy"],
    },
  },
}
```

在 `allowlist` 模式下，你也可以通过添加显式的 `groups.<chat_id>` 条目来接纳一个群。显式条目不会覆盖 `groupPolicy: "disabled"`。`groups.*` 下的通配符默认值配置匹配的群，但它们本身不会接纳群。

```json5
{
  channels: {
    feishu: {
      groupPolicy: "allowlist",
      groups: {
        oc_xxx: {
          requireMention: false,
        },
      },
    },
  },
}
```

### 限制群内的发送者

```json5
{
  channels: {
    feishu: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["oc_xxx"],
      groups: {
        oc_xxx: {
          // 用户 open_id 格式如：ou_xxx
          allowFrom: ["ou_user1", "ou_user2"],
        },
      },
    },
  },
}
```

---

<a id="get-groupuser-ids"></a>

## 获取群/用户 ID

### 群 ID（`chat_id`，格式：`oc_xxx`）

在飞书/Lark 中打开群，点击右上角的菜单图标，进入**设置**。群 ID（`chat_id`）列在设置页面上。

![获取群 ID](/images/feishu-get-group-id.png)

### 用户 ID（`open_id`，格式：`ou_xxx`）

启动 Gateway，给机器人发一条私聊消息，然后查看日志：

```bash
openclaw logs --follow
```

在日志输出中查找 `open_id`。你也可以检查待处理的配对请求：

```bash
openclaw pairing list feishu
```

---

## 常用命令

| 命令 | 描述 |
| --- | --- |
| `/status` | 显示机器人状态 |
| `/reset` | 重置当前会话 |
| `/model` | 显示或切换 AI 模型 |

<Note>
飞书/Lark 不支持原生斜杠命令菜单，因此请以纯文本消息形式发送这些命令。
</Note>

---

## 故障排除

### 机器人在群聊中不响应

1. 确保机器人已被添加到群中
2. 确保你 @了机器人（默认需要）
3. 验证 `groupPolicy` 不是 `"disabled"`
4. 检查日志：`openclaw logs --follow`

### 机器人收不到消息

1. 确保机器人已在飞书开放平台 / Lark Developer 上发布并审核通过
2. 确保事件订阅包含 `im.message.receive_v1`
3. 确保选择了**长连接**（WebSocket）
4. 确保已授予所有必需的权限范围
5. 确保 Gateway 正在运行：`openclaw gateway status`
6. 检查日志：`openclaw logs --follow`

### 二维码设置在飞书手机应用中无反应

1. 重新运行设置：`openclaw channels login --channel feishu`
2. 选择手动设置
3. 在飞书开放平台中创建自建应用并复制其 App ID 和 App Secret
4. 将这些凭据粘贴到设置向导中

### App Secret 泄露

1. 在飞书开放平台 / Lark Developer 中重置 App Secret
2. 在配置中更新该值
3. 重启 Gateway：`openclaw gateway restart`

---

## 高级配置

### 多账号

```json5
{
  channels: {
    feishu: {
      defaultAccount: "main",
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
          name: "主机器人",
          tts: {
            providers: {
              openai: { voice: "shimmer" },
            },
          },
        },
        backup: {
          appId: "cli_yyy",
          appSecret: "yyy",
          name: "备用机器人",
          enabled: false,
        },
      },
    },
  },
}
```

`defaultAccount` 控制出站 API 在未指定 `accountId` 时使用哪个账号。`accounts.<id>.tts` 使用与 `messages.tts` 相同的结构，并在全局 TTS 配置之上深度合并，因此多机器人飞书设置可以在全局共享提供商凭据，同时仅按账号覆盖语音、模型、人设或自动模式。

### 消息限制

- `textChunkLimit` - 出站文本分块大小（默认：`2000` 字符）
- `mediaMaxMb` - 媒体上传/下载限制（默认：`30` MB）

### 流式输出

飞书/Lark 支持通过交互式卡片流式回复。启用后，机器人生成文本时会实时更新卡片。

```json5
{
  channels: {
    feishu: {
      streaming: true, // 启用流式卡片输出（默认：true）
      blockStreaming: true, // 选择加入已完成块的流式传输
    },
  },
}
```

设置 `streaming: false` 可以一次性发送完整回复。`blockStreaming` 默认关闭；仅当你希望在最终回复之前刷新已完成的助手块时才启用它。

### 配额优化

使用两个可选标志来减少飞书/Lark API 调用次数：

- `typingIndicator`（默认 `true`）：设置为 `false` 以跳过打字反应调用
- `resolveSenderNames`（默认 `true`）：设置为 `false` 以跳过发送者资料查找

```json5
{
  channels: {
    feishu: {
      typingIndicator: false,
      resolveSenderNames: false,
    },
  },
}
```

### ACP 会话

飞书/Lark 支持 DM 和群话题消息的 ACP。飞书/Lark ACP 是文本命令驱动的——没有原生斜杠命令菜单，因此直接在对话中使用 `/acp ...` 消息。

#### 持久 ACP 绑定

```json5
{
  agents: {
    list: [
      {
        id: "codex",
        runtime: {
          type: "acp",
          acp: {
            agent: "codex",
            backend: "acpx",
            mode: "persistent",
            cwd: "/workspace/openclaw",
          },
        },
      },
    ],
  },
  bindings: [
    {
      type: "acp",
      agentId: "codex",
      match: {
        channel: "feishu",
        accountId: "default",
        peer: { kind: "direct", id: "ou_1234567890" },
      },
    },
    {
      type: "acp",
      agentId: "codex",
      match: {
        channel: "feishu",
        accountId: "default",
        peer: { kind: "group", id: "oc_group_chat:topic:om_topic_root" },
      },
      acp: { label: "codex-feishu-topic" },
    },
  ],
}
```

#### 从聊天中生成 ACP

在飞书/Lark 的 DM 或话题中：

```text
/acp spawn codex --thread here
```

`--thread here` 适用于 DM 和飞书/Lark 话题消息。绑定对话中的后续消息会直接路由到该 ACP 会话。

### 多代理路由

使用 `bindings` 将飞书/Lark 的 DM 或群路由到不同的代理。

```json5
{
  agents: {
    list: [
      { id: "main" },
      { id: "agent-a", workspace: "/home/user/agent-a" },
      { id: "agent-b", workspace: "/home/user/agent-b" },
    ],
  },
  bindings: [
    {
      agentId: "agent-a",
      match: {
        channel: "feishu",
        peer: { kind: "direct", id: "ou_xxx" },
      },
    },
    {
      agentId: "agent-b",
      match: {
        channel: "feishu",
        peer: { kind: "group", id: "oc_zzz" },
      },
    },
  ],
}
```

路由字段：

- `match.channel`: `"feishu"`
- `match.peer.kind`: `"direct"`（私聊）或 `"group"`（群聊）
- `match.peer.id`: 用户 Open ID（`ou_xxx`）或群 ID（`oc_xxx`）

查找方法参见[获取群/用户 ID](#get-groupuser-ids)。

---

## 配置参考

完整配置：[Gateway 配置](/zh-CN/gateway/configuration)

| 设置 | 描述 | 默认值 |
| --- | --- | --- |
| `channels.feishu.enabled` | 启用/禁用通道 | `true` |
| `channels.feishu.domain` | API 域名（`feishu` 或 `lark`） | `feishu` |
| `channels.feishu.connectionMode` | 事件传输（`websocket` 或 `webhook`） | `websocket` |
| `channels.feishu.defaultAccount` | 出站路由的默认账号 | `default` |
| `channels.feishu.verificationToken` | webhook 模式必需 | - |
| `channels.feishu.encryptKey` | webhook 模式必需 | - |
| `channels.feishu.webhookPath` | Webhook 路由路径 | `/feishu/events` |
| `channels.feishu.webhookHost` | Webhook 绑定主机 | `127.0.0.1` |
| `channels.feishu.webhookPort` | Webhook 绑定端口 | `3000` |
| `channels.feishu.accounts.<id>.appId` | App ID | - |
| `channels.feishu.accounts.<id>.appSecret` | App Secret | - |
| `channels.feishu.accounts.<id>.domain` | 每个账号的域名覆盖 | `feishu` |
| `channels.feishu.accounts.<id>.tts` | 每个账号的 TTS 覆盖 | `messages.tts` |
| `channels.feishu.dmPolicy` | DM 策略 | `allowlist` |
| `channels.feishu.allowFrom` | DM 允许列表（open_id 列表） | [BotOwnerId] |
| `channels.feishu.groupPolicy` | 群策略 | `allowlist` |
| `channels.feishu.groupAllowFrom` | 群允许列表 | - |
| `channels.feishu.requireMention` | 群中需要 @提及 | `true` |
| `channels.feishu.groups.<chat_id>.requireMention` | 每个群的 @提及覆盖；显式 ID 也在 allowlist 模式下接纳该群 | 继承 |
| `channels.feishu.groups.<chat_id>.enabled` | 启用/禁用特定的群 | `true` |
| `channels.feishu.textChunkLimit` | 消息分块大小 | `2000` |
| `channels.feishu.mediaMaxMb` | 媒体大小限制 | `30` |
| `channels.feishu.streaming` | 流式卡片输出 | `true` |
| `channels.feishu.blockStreaming` | 已完成块的回复流式传输 | `false` |
| `channels.feishu.typingIndicator` | 发送打字反应 | `true` |
| `channels.feishu.resolveSenderNames` | 解析发送者显示名称 | `true` |

---

## 支持的消息类型

### 接收

- ✅ 文本
- ✅ 富文本（post）
- ✅ 图片
- ✅ 文件
- ✅ 音频
- ✅ 视频/媒体
- ✅ 贴纸

入站飞书/Lark 音频消息被规范化为媒体占位符，而不是原始 `file_key` JSON。当配置了 `tools.media.audio` 时，OpenClaw 会下载语音笔记资源并在代理回合之前运行共享的音频转录，因此代理会收到语音转录文本。如果飞书直接在音频负载中包含转录文本，则该文本会被使用而无需另一次 ASR 调用。没有音频转录提供商时，代理仍然会收到 `<media:audio>` 占位符以及保存的附件，而不是原始飞书资源负载。

### 发送

- ✅ 文本
- ✅ 图片
- ✅ 文件
- ✅ 音频
- ✅ 视频/媒体
- ✅ 交互式卡片（包括流式更新）
- ⚠️ 富文本（post 风格的排版；不支持完整的飞书/Lark 创作功能）

原生飞书/Lark 音频气泡使用飞书 `audio` 消息类型，需要上传 Ogg/Opus 媒体（`file_type: "opus"`）。现有的 `.opus` 和 `.ogg` 媒体会直接作为原生音频发送。MP3/WAV/M4A 和其他可能的音频格式仅在回复请求语音投递（`audioAsVoice` / 消息工具 `asVoice`，包括 TTS 语音笔记回复）时通过 `ffmpeg` 转码为 48kHz Ogg/Opus。普通 MP3 附件保持为常规文件。如果缺少 `ffmpeg` 或转换失败，OpenClaw 会回退到文件附件并记录原因。

### 话题和回复

- ✅ 内联回复
- ✅ 话题回复
- ✅ 回复话题消息时，媒体回复保持话题感知

对于 `groupSessionScope: "group_topic"` 和 `"group_topic_sender"`，原生飞书/Lark 话题群使用事件 `thread_id`（`omt_*`）作为规范的话题会话键。如果原生话题启动器事件缺少 `thread_id`，OpenClaw 会在路由回合之前从飞书获取它。OpenClaw 转为话题的普通群回复继续使用回复根消息 ID（`om_*`），以便第一轮和后续轮保持在同一个会话中。

---

## 相关文档

- [通道概览](/zh-CN/channels) - 所有支持的通道
- [配对](/zh-CN/channels/pairing) - DM 认证和配对流程
- [群组](/zh-CN/channels/groups) - 群聊行为和提及门控
- [通道路由](/zh-CN/channels/channel-routing) - 消息的会话路由
- [安全](/zh-CN/gateway/security) - 访问模型和加固
