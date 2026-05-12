---
summary: "CLI 引导向导：网关、工作区、通道和技能的引导式设置"
read_when:
  - 运行或配置 CLI 引导向导时
  - 设置新机器时
title: "引导向导（CLI）"
sidebarTitle: "引导向导：CLI"
---

CLI 引导向导是在 macOS、Linux 或 Windows（通过 WSL2；强烈推荐）上设置 OpenClaw 的**推荐**方式。它在一个引导流程中配置本地 Gateway 或远程 Gateway 连接，以及通道、技能和工作区默认值。

```bash
openclaw onboard
```

<Info>
最快的首次聊天：打开控制面板（无需设置通道）。运行 `openclaw dashboard`，在浏览器中聊天。文档：[控制面板](/zh-CN/web/dashboard)。
</Info>

稍后重新配置：

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` 不代表非交互模式。脚本请使用 `--non-interactive`。
</Note>

<Tip>
CLI 引导向导包含一个网页搜索步骤，你可以选择提供商，如 Brave、DuckDuckGo、Exa、Firecrawl、Gemini、Grok、Kimi、MiniMax Search、Ollama Web Search、Perplexity、SearXNG 或 Tavily。部分提供商需要 API 密钥，其他则不需要。你也可以稍后用 `openclaw configure --section web` 配置。文档：[网页工具](/zh-CN/tools/web)。
</Tip>

## 快速开始 vs 高级

引导向导以**快速开始**（默认值）或**高级**（完全控制）开始。

<Tabs>
  <Tab title="快速开始（默认值）">
    - 本地 Gateway（回环）
    - 工作区默认值（或现有工作区）
    - Gateway 端口 **18789**
    - Gateway 认证 **Token**（自动生成，即使是回环）
    - 新本地设置的工具策略默认值：`tools.profile: "coding"`（保留现有显式策略）
    - DM 隔离默认值：本地引导在未设置时写入 `session.dmScope: "per-channel-peer"`。详情：[CLI 设置参考](/zh-CN/start/wizard-cli-reference#outputs-and-internals)
    - Tailscale 暴露 **关闭**
    - Telegram + WhatsApp DM 默认为**允许列表**（会提示输入你的手机号）

  </Tab>
  <Tab title="高级（完全控制）">
    - 暴露每一步（模式、工作区、网关、通道、守护进程、技能）。

  </Tab>
</Tabs>

## 引导向导配置的内容

**本地模式（默认）**会引导你完成以下步骤：

1. **模型/认证** — 选择任何支持的提供商/认证流程（API 密钥、OAuth 或提供商特定的手动认证），包括自定义提供商（兼容 OpenAI、兼容 Anthropic 或未知自动检测）。选择默认模型。
   安全提示：如果此代理将运行工具或处理 webhook/hooks 内容，请使用最新的新一代最强模型，并保持严格的工具策略。较弱/较旧的层级更容易受到提示注入攻击。
   对于非交互运行，`--secret-input-mode ref` 在认证配置文件中存储环境变量支持的引用，而不是明文 API 密钥值。
   在非交互 `ref` 模式下，必须设置提供商环境变量；传递内联密钥标志而不设置该环境变量会快速失败。
   在交互运行中，选择密钥引用模式允许你指向环境变量或配置的提供商引用（`file` 或 `exec`），在保存前进行快速预检验证。
   对于 Anthropic，交互引导/配置提供 **Anthropic Claude CLI** 作为首选本地路径和 **Anthropic API 密钥** 作为推荐的生产路径。Anthropic 设置令牌也可作为支持的令牌认证路径。
2. **工作区** — 代理文件的位置（默认 `~/.openclaw/workspace`）。播种引导文件。
3. **Gateway** — 端口、绑定地址、认证模式、Tailscale 暴露。
   在交互令牌模式下，选择默认明文令牌存储或选择使用 SecretRef。
   非交互令牌 SecretRef 路径：`--gateway-token-ref-env <ENV_VAR>`。
4. **通道** — 内置和捆绑的聊天通道，如 iMessage、Discord、飞书、Google Chat、Mattermost、Microsoft Teams、QQ Bot、Signal、Slack、Telegram、WhatsApp 等。
5. **守护进程** — 安装 LaunchAgent（macOS）、systemd 用户单元（Linux/WSL2）或原生 Windows 计划任务，以及每用户启动文件夹回退。
   如果令牌认证需要令牌且 `gateway.auth.token` 由 SecretRef 管理，守护进程安装会验证它但不会将解析的令牌持久化到监督服务环境元数据中。
   如果令牌认证需要令牌且配置的令牌 SecretRef 未解析，守护进程安装会被阻止并提供可操作的指导。
   如果同时配置了 `gateway.auth.token` 和 `gateway.auth.password` 且 `gateway.auth.mode` 未设置，守护进程安装会在显式设置模式之前被阻止。
6. **健康检查** — 启动 Gateway 并验证其正在运行。
7. **技能** — 安装推荐的技能和可选依赖。

<Note>
重新运行引导向导**不会**清空任何内容，除非你明确选择**重置**（或传递 `--reset`）。
CLI `--reset` 默认为配置、凭据和会话；使用 `--reset-scope full` 包含工作区。
如果配置无效或包含遗留密钥，引导向导会要求你先运行 `openclaw doctor`。
</Note>

**远程模式**仅配置本地客户端以连接到其他地方的 Gateway。它**不会**在远程主机上安装或更改任何内容。

## 添加另一个代理

使用 `openclaw agents add <name>` 创建一个独立的代理，拥有自己的工作区、会话和认证配置文件。不带 `--workspace` 运行会启动引导向导。

配置内容：

- `agents.list[].name`
- `agents.list[].workspace`
- `agents.list[].agentDir`

注意：

- 默认工作区遵循 `~/.openclaw/workspace-<agentId>`。
- 添加 `bindings` 以路由入站消息（引导向导可以完成此操作）。
- 非交互标志：`--model`、`--agent-dir`、`--bind`、`--non-interactive`。

## 完整参考

有关详细的逐步分解和配置输出，请参阅
[CLI 设置参考](/zh-CN/start/wizard-cli-reference)。
有关非交互示例，请参阅 [CLI 自动化](/zh-CN/start/wizard-cli-automation)。
有关更深入的技术参考，包括 RPC 细节，请参阅
[引导向导参考](/zh-CN/reference/wizard)。

## 相关文档

- CLI 命令参考：[`openclaw onboard`](/zh-CN/cli/onboard)
- 引导向导概览：[引导向导概览](/zh-CN/start/onboarding-overview)
- macOS 应用引导向导：[引导向导](/zh-CN/start/onboarding)
- 代理首次运行仪式：[代理引导](/zh-CN/start/bootstrapping)
