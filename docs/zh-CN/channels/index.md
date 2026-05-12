---
summary: "OpenClaw 可以连接的消息平台"
read_when:
  - 你想为 OpenClaw 选择一个聊天通道时
  - 你需要快速了解支持的消息平台时
title: "聊天通道"
---

OpenClaw 可以在你已经在使用的任何聊天应用上与你对话。每个通道都通过 Gateway 连接。文本在所有通道上都受支持；媒体和反应功能因通道而异。

## 投递说明

- 包含 markdown 图片语法的 Telegram 回复（如 `![alt](url)`）在最终出站路径上会尽可能转换为媒体回复。
- Slack 多人 DM 作为群聊路由，因此群策略、提及行为和群会话规则适用于 MPIM 对话。
- WhatsApp 设置是按需安装的：引导向导可以在插件包安装之前显示设置流程，Gateway 仅在通道实际激活时才加载 WhatsApp 运行时。

## 支持的通道

- [Discord](/zh-CN/channels/discord) - Discord Bot API + Gateway；支持服务器、频道和 DM。
- [飞书](/zh-CN/channels/feishu) - 通过 WebSocket 的飞书/Lark 机器人（捆绑插件）。
- [Google Chat](/zh-CN/channels/googlechat) - 通过 HTTP webhook 的 Google Chat API 应用（可下载插件）。
- [iMessage](/zh-CN/channels/imessage) - 通过已登录 Mac 上的 `imsg` 桥接进行原生 macOS 集成（或当 Gateway 运行在其他地方时使用 SSH 包装器），包括用于回复、tapback、特效、附件和群组管理的私有 API 操作。当主机权限和 Messages 访问匹配时，是新的 OpenClaw iMessage 设置的首选。
- [IRC](/zh-CN/channels/irc) - 经典 IRC 服务器；频道 + DM，支持配对/允许列表控制。
- [LINE](/zh-CN/channels/line) - LINE Messaging API 机器人（可下载插件）。
- [Matrix](/zh-CN/channels/matrix) - Matrix 协议（可下载插件）。
- [Mattermost](/zh-CN/channels/mattermost) - Bot API + WebSocket；频道、群组、DM（可下载插件）。
- [Microsoft Teams](/zh-CN/channels/msteams) - Bot Framework；企业支持（捆绑插件）。
- [Nextcloud Talk](/zh-CN/channels/nextcloud-talk) - 通过 Nextcloud Talk 的自托管聊天（捆绑插件）。
- [Nostr](/zh-CN/channels/nostr) - 通过 NIP-04 的去中心化 DM（捆绑插件）。
- [QQ Bot](/zh-CN/channels/qqbot) - QQ Bot API；私聊、群聊和富媒体（捆绑插件）。
- [Signal](/zh-CN/channels/signal) - signal-cli；注重隐私。
- [Slack](/zh-CN/channels/slack) - Bolt SDK；工作区应用。
- [Synology Chat](/zh-CN/channels/synology-chat) - 通过出站+入站 webhooks 的 Synology NAS Chat（捆绑插件）。
- [Telegram](/zh-CN/channels/telegram) - 通过 grammY 的 Bot API；支持群组。
- [Tlon](/zh-CN/channels/tlon) - 基于 Urbit 的即时通讯（捆绑插件）。
- [Twitch](/zh-CN/channels/twitch) - 通过 IRC 连接的 Twitch 聊天（捆绑插件）。
- [语音通话](/zh-CN/plugins/voice-call) - 通过 Plivo 或 Twilio 的电话（插件，单独安装）。
- [WebChat](/zh-CN/web/webchat) - Gateway WebChat UI，通过 WebSocket。
- [微信](/zh-CN/channels/wechat) - 腾讯 iLink Bot 插件，通过二维码登录；仅限私聊（外部插件）。
- [WhatsApp](/zh-CN/channels/whatsapp) - 最流行的；使用 Baileys，需要二维码配对。
- [元宝](/zh-CN/channels/yuanbao) - 腾讯元宝机器人（外部插件）。
- [Zalo](/zh-CN/channels/zalo) - Zalo Bot API；越南流行的即时通讯（捆绑插件）。
- [Zalo 个人](/zh-CN/channels/zalouser) - 通过二维码登录的 Zalo 个人账户（捆绑插件）。

## 注意

- 通道可以同时运行；配置多个，OpenClaw 将按聊天路由。
- 通常设置最快的是 **Telegram**（简单的机器人令牌）。WhatsApp 需要二维码配对并在磁盘上存储更多状态。
- 群行为因通道而异；查看[群组](/zh-CN/channels/groups)。
- DM 配对和允许列表为了安全而强制执行；查看[安全](/zh-CN/gateway/security)。
- 故障排除：[通道故障排除](/zh-CN/channels/troubleshooting)。
- 模型提供商在单独文档中；查看[模型提供商](/zh-CN/providers/models)。
