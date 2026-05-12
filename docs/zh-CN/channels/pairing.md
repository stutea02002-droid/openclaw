---
summary: "配对概览：批准谁可以给你的机器人发 DM + 哪些节点可以加入"
read_when:
  - 设置 DM 访问控制时
  - 配对新的 iOS/Android 节点时
  - 审查 OpenClaw 安全态势时
title: "配对"
---

"配对"是 OpenClaw 的显式访问批准步骤。它用于两个地方：

1. **DM 配对**（谁被允许与机器人对话）
2. **节点配对**（哪些设备/节点被允许加入 Gateway 网络）

安全上下文：[安全](/zh-CN/gateway/security)

## 1) DM 配对（入站聊天访问）

当通道配置为 DM 策略 `pairing` 时，未知发送者会收到一个短码，他们的消息在获得批准前**不会被处理**。

默认 DM 策略记录在：[安全](/zh-CN/gateway/security)

`dmPolicy: "open"` 仅在有效 DM 允许列表包含 `"*"` 时才是公开的。设置和验证需要通配符用于公开开放配置。如果现有状态包含 `open` 带有具体的 `allowFrom` 条目，运行时仍然只允许那些发送者，配对存储批准不会扩大 `open` 访问。

配对码：

- 8 个字符，大写字母，无歧义字符（`0O1I`）。
- **1 小时后过期**。机器人仅在创建新请求时发送配对消息（每个发送者大约每小时一次）。
- 待处理的 DM 配对请求默认每个通道最多 **3 个**；在某个请求过期或被批准之前，额外的请求会被忽略。

### 批准发送者

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

如果尚未配置命令所有者，批准 DM 配对码还会将 `commands.ownerAllowFrom` 引导到批准的发送者，例如 `telegram:123456789`。这为首次设置提供了一个显式的所有者，用于特权命令和执行批准提示。在所有者存在之后，后续的配对批准只授予 DM 访问权限；不会添加更多所有者。

支持的通道：`discord`、`feishu`、`googlechat`、`imessage`、`irc`、`line`、`matrix`、`mattermost`、`msteams`、`nextcloud-talk`、`nostr`、`openclaw-weixin`、`signal`、`slack`、`synology-chat`、`telegram`、`twitch`、`whatsapp`、`zalo`、`zalouser`。

### 可复用的发送者组

当相同的受信任发送者集应该应用于多个消息通道或同时应用于 DM 和群允许列表时，使用顶层 `accessGroups`。

静态组使用 `type: "message.senders"`，并通过通道允许列表中的 `accessGroup:<name>` 引用：

```json5
{
  accessGroups: {
    operators: {
      type: "message.senders",
      members: {
        discord: ["discord:123456789012345678"],
        telegram: ["987654321"],
        whatsapp: ["+15551234567"],
      },
    },
  },
  channels: {
    telegram: { dmPolicy: "allowlist", allowFrom: ["accessGroup:operators"] },
    whatsapp: { groupPolicy: "allowlist", groupAllowFrom: ["accessGroup:operators"] },
  },
}
```

访问组的详细文档在此处：[访问组](/zh-CN/channels/access-groups)

### 状态存储在哪里

存储在 `~/.openclaw/credentials/` 下：

- 待处理请求：`<channel>-pairing.json`
- 批准的允许列表存储：
  - 默认账号：`<channel>-allowFrom.json`
  - 非默认账号：`<channel>-<accountId>-allowFrom.json`

账号范围行为：

- 非默认账号仅读写其范围化的允许列表文件。
- 默认账号使用通道范围的非范围化允许列表文件。

将这些视为敏感信息（它们控制对你的助手的访问）。

<Note>
配对允许列表存储用于 DM 访问。群组授权是独立的。批准 DM 配对码不会自动允许该发送者在群组中运行命令或控制机器人。首次所有者引导是 `commands.ownerAllowFrom` 中的独立配置状态，群聊交付仍然遵循通道的群允许列表（例如 `groupAllowFrom`、`groups`，或每个群/每个话题的覆盖，具体取决于通道）。
</Note>

## 2) 节点设备配对（iOS/Android/macOS/无头节点）

节点作为**设备**连接到 Gateway，角色为 `role: node`。Gateway 会创建设备配对请求，必须被批准。

### 通过 Telegram 配对（iOS 推荐）

如果你使用 `device-pair` 插件，你可以完全从 Telegram 进行首次设备配对：

1. 在 Telegram 中，给机器人发消息：`/pair`
2. 机器人回复两条消息：一条说明消息和一条单独的**设置码**消息（易于在 Telegram 中复制/粘贴）。
3. 在手机上，打开 OpenClaw iOS 应用 → 设置 → Gateway。
4. 扫描二维码或粘贴设置码并连接。
5. 回到 Telegram：`/pair pending`（查看请求 ID、角色和范围），然后批准。

设置码是一个 base64 编码的 JSON 负载，包含：

- `url`：Gateway WebSocket URL（`ws://...` 或 `wss://...`）
- `bootstrapToken`：用于初始配对握手的短期单设备引导令牌

该引导令牌携带内置的配对引导配置文件：

- 主要移交的 `node` 令牌保持 `scopes: []`
- 任何移交的 `operator` 令牌保持绑定到引导允许列表：`operator.approvals`、`operator.read`、`operator.talk.secrets`、`operator.write`
- 引导范围检查是角色前缀的，不是一个扁平的范围池：operator 范围条目仅满足 operator 请求，非 operator 角色必须仍然在其自己的角色前缀下请求范围
- 后续令牌轮换/撤销仍然受设备的批准角色合约和调用者会话的 operator 范围限制

在设置码有效期内，像密码一样对待它。

对于 Tailscale、公共或其他远程移动配对，使用 Tailscale Serve/Funnel 或其他 `wss://` Gateway URL。明文 `ws://` 设置码仅对回环、私有 LAN 地址、`.local` Bonjour 主机和 Android 模拟器主机接受。Tailnet CGNAT 地址、`.ts.net` 名称和公共主机仍然会在 QR/设置码发布之前关闭。

### 批准节点设备

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
```

当显式批准被拒绝是因为批准配对设备会话是以仅配对范围打开的，CLI 会使用 `operator.admin` 重试相同的请求。这使得具有管理员能力的已配对设备可以恢复新的 Control UI/浏览器配对，而无需手动编辑 `devices/paired.json`。Gateway 仍然会验证重试的连接；无法使用 `operator.admin` 认证的令牌仍然被阻止。

如果同一设备使用不同的认证详细信息重试（例如不同的角色/范围/公钥），先前的待处理请求会被取代，并创建新的 `requestId`。

<Note>
已经配对的设备不会静默获得更广泛的访问。如果它重新连接请求更多范围或更广泛的角色，OpenClaw 会保持现有批准不变，并创建一个新的待处理升级请求。使用 `openclaw devices list` 在批准之前比较当前批准的访问与新请求的访问。
</Note>

### 可选可信 CIDR 节点自动批准

设备配对默认为手动。对于严格控制的节点网络，你可以选择使用显式 CIDR 或精确 IP 进行首次节点自动批准：

```json5
{
  gateway: {
    nodes: {
      pairing: {
        autoApproveCidrs: ["192.168.1.0/24"],
      },
    },
  },
}
```

这仅适用于没有请求范围的新的 `role: node` 配对请求。Operator、浏览器、Control UI 和 WebChat 客户端仍然需要手动批准。角色、范围、元数据和公钥变更仍然需要手动批准。

### 节点配对状态存储

存储在 `~/.openclaw/devices/` 下：

- `pending.json`（短期；待处理请求会过期）
- `paired.json`（已配对设备 + 令牌）

### 注意

- 遗留的 `node.pair.*` API（CLI：`openclaw nodes pending|approve|reject|remove|rename`）是一个独立的 Gateway 拥有的配对存储。WS 节点仍然需要设备配对。
- 配对记录是已批准角色的持久真相源。活跃设备令牌保持绑定到该批准角色集；批准角色之外的 stray 令牌条目不会创建新访问。

## 相关文档

- 安全模型 + 提示注入：[安全](/zh-CN/gateway/security)
- 安全更新（运行 doctor）：[更新](/zh-CN/install/updating)
- 通道配置：
  - Telegram：[Telegram](/zh-CN/channels/telegram)
  - WhatsApp：[WhatsApp](/zh-CN/whatsapp)
  - Signal：[Signal](/zh-CN/channels/signal)
  - iMessage：[iMessage](/zh-CN/channels/imessage)
  - Discord：[Discord](/zh-CN/channels/discord)
  - Slack：[Slack](/zh-CN/channels/slack)
