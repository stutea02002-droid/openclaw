---
summary: "代理运行时、工作区契约和会话引导"
read_when:
  - 更改代理运行时、工作区引导或会话行为时
title: "代理运行时"
---

OpenClaw 运行一个**单一嵌入式代理运行时**——每个 Gateway 一个代理进程，拥有自己的工作区、引导文件和会话存储。本页涵盖该运行时契约：工作区必须包含什么、哪些文件会被注入，以及会话如何对其进行引导。

## 工作区（必需）

OpenClaw 使用单个代理工作区目录（`agents.defaults.workspace`）作为代理用于工具和上下文的**唯一**工作目录（`cwd`）。

推荐：如果缺少 `~/.openclaw/openclaw.json`，使用 `openclaw setup` 创建它并初始化工作区文件。

完整工作区布局 + 备份指南：[代理工作区](/zh-CN/concepts/agent-workspace)

如果启用了 `agents.defaults.sandbox`，非主会话可以在 `agents.defaults.sandbox.workspaceRoot` 下覆盖为每个会话的工作区（参见[Gateway 配置](/zh-CN/gateway/configuration)）。

## 引导文件（注入）

在 `agents.defaults.workspace` 中，OpenClaw 期望这些用户可编辑的文件：

- `AGENTS.md` - 操作说明 + "记忆"
- `SOUL.md` - 人设、边界、语气
- `TOOLS.md` - 用户维护的工具笔记（如 `imsg`、`sag`、约定）
- `BOOTSTRAP.md` - 一次性首次运行仪式（完成后删除）
- `IDENTITY.md` - 代理名称/风格/表情符号
- `USER.md` - 用户档案 + 偏好称呼

在新会话的第一轮中，OpenClaw 将这些文件的内容注入到系统提示的项目上下文中。

空文件会被跳过。大文件会被修剪并在末尾标记截断标记，以保持提示简洁（读取文件获取完整内容）。

如果文件缺失，OpenClaw 会注入一个"文件缺失"标记行（`openclaw setup` 会创建一个安全的默认模板）。

`BOOTSTRAP.md` 仅为**全新的工作区**（不存在其他引导文件）创建。当它处于待处理状态时，OpenClaw 会将其保留在项目上下文中，并为初始仪式添加系统提示引导指导，而不是将其复制到用户消息中。如果你在完成仪式后删除它，后续的重新启动不应该重新创建它。

要完全禁用引导文件创建（对于预种子工作区），设置：

```json5
{ agents: { defaults: { skipBootstrap: true } } }
```

## 内置工具

核心工具（read/exec/edit/write 及相关系统工具）始终可用，受工具策略约束。`apply_patch` 是可选的，由 `tools.exec.applyPatch` 控制。`TOOLS.md` **不**控制哪些工具存在；它是关于你希望如何**使用**它们的指导。

## 技能

OpenClaw 从以下位置加载技能（优先级从高到低）：

- 工作区：`<workspace>/skills`
- 项目代理技能：`<workspace>/.agents/skills`
- 个人代理技能：`~/.agents/skills`
- 托管/本地：`~/.openclaw/skills`
- 捆绑（随安装一起发布）
- 额外技能文件夹：`skills.load.extraDirs`

技能可以通过配置/环境变量进行门控（参见[Gateway 配置](/zh-CN/gateway/configuration)中的 `skills`）。

## 运行时边界

嵌入式代理运行时基于 Pi 代理核心（模型、工具和提示管道）构建。会话管理、发现、工具连线和通道交付是 OpenClaw 在该核心之上的自有层。

## 会话

会话转录文件以 JSONL 格式存储在：

- `~/.openclaw/agents/<agentId>/sessions/<SessionId>.jsonl`

会话 ID 是稳定的，由 OpenClaw 选择。不读取来自其他工具的遗留会话文件夹。

## 流式 Steering

当队列模式为 `steer` 时，入站消息被注入到当前运行中。排队 steering 会在**当前助手回合完成执行其工具调用之后**、下一次 LLM 调用之前交付。Pi 会为 `steer` 一起排出所有待处理的 steering 消息；遗留的 `queue` 每次模型边界排出一个消息。Steering 不再跳过当前助手消息中剩余的工具调用。

当队列模式为 `followup` 或 `collect` 时，入站消息会被保持直到当前回合结束，然后新的代理回合会使用排队的负载开始。模式与边界行为参见[队列](/zh-CN/concepts/queue)和[Steering 队列](/zh-CN/concepts/queue-steering)。

块流式传输会在助手块完成后立即发送；**默认关闭**（`agents.defaults.blockStreamingDefault: "off"`）。通过 `agents.defaults.blockStreamingBreak` 调整边界（`text_end` 与 `message_end`；默认为 text_end）。使用 `agents.defaults.blockStreamingChunk` 控制软块分块（默认为 800-1200 字符；优先段落分隔，然后是换行；最后是句子）。使用 `agents.defaults.blockStreamingCoalesce` 合并流式块以减少单行突发（发送前基于空闲的合并）。非 Telegram 通道需要显式设置 `*.blockStreaming: true` 才能启用块回复。详细工具摘要在工具启动时发出（无去重）；Control UI 在可用时通过代理事件流式传输工具输出。更多细节：[流式 + 分块](/zh-CN/concepts/streaming)。

## 模型引用

配置中的模型引用（例如 `agents.defaults.model` 和 `agents.defaults.models`）通过按**第一个** `/` 分割来解析。

- 配置模型时使用 `provider/model`。
- 如果模型 ID 本身包含 `/`（OpenRouter 风格），包含提供商前缀（例如：`openrouter/moonshotai/kimi-k2`）。
- 如果省略提供商，OpenClaw 会先尝试别名，然后为该确切模型 ID 匹配唯一已配置的提供商，最后才回退到配置的默认提供商。如果该提供商不再暴露配置的默认模型，OpenClaw 会回退到第一个配置的提供商/模型，而不是暴露一个已移除的过时默认值。

## 配置（最小）

至少设置：

- `agents.defaults.workspace`
- `channels.whatsapp.allowFrom`（强烈推荐）

---

_下一节：[群聊](/zh-CN/channels/group-messages)_ 🦞

## 相关文档

- [代理工作区](/zh-CN/concepts/agent-workspace)
- [多代理路由](/zh-CN/concepts/multi-agent)
- [会话管理](/zh-CN/concepts/session)
