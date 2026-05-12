---
summary: "OpenClaw 在通道、路由、媒体和用户体验方面的能力。"
read_when:
  - 你想获取 OpenClaw 支持的功能完整列表
title: "功能"
---

## 亮点

<Columns>
  <Card title="通道" icon="message-square" href="/zh-CN/channels">
    一个 Gateway 即可支持 Discord、iMessage、Signal、Slack、Telegram、WhatsApp、WebChat 等。
  </Card>
  <Card title="插件" icon="plug" href="/zh-CN/tools/plugin">
    捆绑插件在正式版本中添加 Matrix、Nextcloud Talk、Nostr、Twitch、Zalo 等支持，无需额外安装。
  </Card>
  <Card title="路由" icon="route" href="/zh-CN/concepts/multi-agent">
    多代理路由，隔离会话。
  </Card>
  <Card title="媒体" icon="image" href="/zh-CN/nodes/images">
    图片、音频、视频、文档，以及图片和视频生成。
  </Card>
  <Card title="应用与界面" icon="monitor" href="/zh-CN/web/control-ui">
    Web 控制面板和 macOS 伴侣应用。
  </Card>
  <Card title="移动节点" icon="smartphone" href="/zh-CN/nodes">
    iOS 和 Android 节点，支持配对、语音/聊天，以及丰富的设备命令。
  </Card>
</Columns>

## 完整列表

**通道：**

- 内置通道包括 Discord、Google Chat、iMessage、IRC、Signal、Slack、Telegram、WebChat 和 WhatsApp
- 捆绑插件通道包括飞书、LINE、Matrix、Mattermost、Microsoft Teams、Nextcloud Talk、Nostr、QQ Bot、Synology Chat、Tlon、Twitch、Zalo 和 Zalo Personal
- 可选的单独安装通道插件包括 Voice Call 和第三方软件包（如微信）
- 第三方通道插件可以进一步扩展 Gateway，例如微信
- 群聊支持，基于提及的激活方式
- 私聊安全保护，包含允许列表和配对机制

**代理：**

- 嵌入式代理运行时，支持工具流式调用
- 多代理路由，按工作区或发送者隔离会话
- 会话：私聊合并到共享的 `main`；群组隔离
- 长回复支持流式和分段

**认证与提供商：**

- 35+ 模型提供商（Anthropic、OpenAI、Google 等）
- 通过 OAuth 的订阅认证（如 OpenAI Codex）
- 自定义和自托管提供商支持（vLLM、SGLang、Ollama，以及任何兼容 OpenAI 或 Anthropic 的端点）

**媒体：**

- 输入和输出图片、音频、视频和文档
- 共享图片生成和视频生成功能界面
- 语音笔记转录
- 文本转语音，支持多个提供商

**应用与界面：**

- WebChat 和浏览器控制面板
- macOS 菜单栏伴侣应用
- iOS 节点，支持配对、Canvas、相机、屏幕录制、位置和语音
- Android 节点，支持配对、聊天、语音、Canvas、相机和设备命令

**工具与自动化：**

- 浏览器自动化、执行、沙箱
- 网页搜索（Brave、DuckDuckGo、Exa、Firecrawl、Gemini、Grok、Kimi、MiniMax Search、Ollama Web Search、Perplexity、SearXNG、Tavily）
- Cron 任务和心跳调度
- 技能、插件和工作流管道（Lobster）

## 相关文档

<CardGroup cols={2}>
  <Card title="实验性功能" href="/zh-CN/concepts/experimental-features" icon="flask">
    尚未发布到默认界面的可选功能。
  </Card>
  <Card title="代理运行时" href="/zh-CN/concepts/agent" icon="robot">
    代理运行时模型以及运行的调度方式。
  </Card>
  <Card title="通道" href="/zh-CN/channels" icon="message-square">
    从一个 Gateway 连接 Telegram、WhatsApp、Discord、Slack 等。
  </Card>
  <Card title="插件" href="/zh-CN/tools/plugin" icon="plug">
    扩展 OpenClaw 的捆绑插件和第三方插件。
  </Card>
</CardGroup>
