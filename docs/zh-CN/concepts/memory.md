---
summary: "OpenClaw 如何跨会话记住事情"
title: "记忆概览"
read_when:
  - 你想了解记忆如何工作时
  - 你想知道要写哪些记忆文件时
---

OpenClaw 通过在代理工作区中写入**纯 Markdown 文件**来记住事情。模型只"记住"保存到磁盘的内容——没有隐藏状态。

## 工作原理

你的代理有三个与记忆相关的文件：

- **`MEMORY.md`** —— 长期记忆。持久的事实、偏好和决策。在每个 DM 会话开始时加载。
- **`memory/YYYY-MM-DD.md`** —— 每日笔记。运行中的上下文和观察。今天和昨天的笔记会自动加载。
- **`DREAMS.md`**（可选）—— 梦境日记和梦境扫描摘要，供人类审查，包括已回填的历史条目。

这些文件位于代理工作区中（默认 `~/.openclaw/workspace`）。

## 各自放什么

`MEMORY.md` 是紧凑的、精心整理的层。用于持久的事实、偏好、常规决策和应在主私人会话开始时可用的简短摘要。它不应该是原始转录、日志或详尽的档案。

`memory/YYYY-MM-DD.md` 文件是工作层。用于详细的每日笔记、观察、会话摘要和以后可能仍有用的原始上下文。这些文件会被 `memory_search` 和 `memory_get` 索引，但不会在每次回合的正常引导提示中注入。

随着时间的推移，代理应该从每日笔记中提取有用的材料到 `MEMORY.md`，并删除过时的长期条目。生成的工作区指令和心跳流程可以定期执行此操作；你不需要为每个记住的细节手动编辑 `MEMORY.md`。

如果 `MEMORY.md` 增长超过引导文件预算，OpenClaw 会在磁盘上保持文件完整，但会截断注入到模型上下文中的副本。将此视为一个信号，将详细材料移回 `memory/*.md`，仅在 `MEMORY.md` 中保留持久摘要，或者如果你明确想花费更多提示预算，可以提高引导限制。使用 `/context list`、`/context detail` 或 `openclaw doctor` 查看原始大小与注入大小以及截断状态。

<Tip>
如果你想让代理记住某件事，直接告诉它就行："记住我更喜欢 TypeScript。"它会把它写到适当的文件中。
</Tip>

## 推断的承诺

一些后续跟进不是持久的事实。如果你提到明天有一个面试，有用的记忆可能是"面试后跟进"，而不是"永久存储在 `MEMORY.md` 中"。

[承诺](/zh-CN/concepts/commitments)是针对这种情况的可选短期后续记忆。OpenClaw 在隐藏的后台过程中推断它们，将它们范围限定到相同的代理和通道，并通过心跳传递到期的检查。显式提醒仍然使用[定时任务](/zh-CN/automation/cron-jobs)。

## 记忆工具

代理有两个用于记忆的工具：

- **`memory_search`** —— 使用语义搜索查找相关笔记，即使措辞与原始内容不同。
- **`memory_get`** —— 读取特定的记忆文件或行范围。

这两个工具由活动的记忆插件（默认：`memory-core`）提供。

## 记忆 Wiki 伴侣插件

如果你希望持久记忆的行为更像维护的知识库而不仅仅是原始笔记，使用捆绑的 `memory-wiki` 插件。

`memory-wiki` 将持久知识编译为 wiki 存储库，包含：

- 确定性页面结构
- 结构化声明和证据
- 矛盾和新鲜度跟踪
- 生成的仪表板
- 为代理/运行时消费者编译的摘要
- wiki 原生工具如 `wiki_search`、`wiki_get`、`wiki_apply` 和 `wiki_lint`

它不会替换活动的记忆插件。活动的记忆插件仍然拥有召回、提升和梦境。`memory-wiki` 在其旁边添加了一个来源丰富的知识层。

参见[记忆 Wiki](/zh-CN/plugins/memory-wiki)。

## 记忆搜索

当配置了嵌入提供商时，`memory_search` 使用**混合搜索**——结合向量相似性（语义含义）与关键字匹配（精确术语如 ID 和代码符号）。一旦你有了任何受支持提供商的 API 密钥，它就可以开箱即用。

<Info>
OpenClaw 从可用的 API 密钥中自动检测你的嵌入提供商。如果你配置了 OpenAI、Gemini、Voyage 或 Mistral 密钥，记忆搜索会自动启用。
</Info>

有关搜索如何工作、调优选项和提供商设置的详细信息，参见[记忆搜索](/zh-CN/concepts/memory-search)。

## 记忆后端

<CardGroup cols={3}>
<Card title="内置（默认）" icon="database" href="/zh-CN/concepts/memory-builtin">
基于 SQLite。开箱即用，支持关键字搜索、向量相似性和混合搜索。无额外依赖。
</Card>
<Card title="QMD" icon="search" href="/zh-CN/concepts/memory-qmd">
本地优先的侧车，具有重新排序、查询扩展和索引工作区外目录的能力。
</Card>
<Card title="Honcho" icon="brain" href="/zh-CN/concepts/memory-honcho">
AI 原生的跨会话记忆，具有用户建模、语义搜索和多代理感知。插件安装。
</Card>
<Card title="LanceDB" icon="layers" href="/zh-CN/plugins/memory-lancedb">
捆绑的 LanceDB 支持的记忆，具有 OpenAI 兼容的嵌入、自动召回、自动捕获和本地 Ollama 嵌入支持。
</Card>
</CardGroup>

## 知识 wiki 层

<CardGroup cols={1}>
<Card title="记忆 Wiki" icon="book" href="/zh-CN/plugins/memory-wiki">
将持久记忆编译为来源丰富的 wiki 存储库，包含声明、仪表板、桥接模式和 Obsidian 友好的工作流。
</Card>
</CardGroup>

## 自动记忆刷新

在[压缩](/zh-CN/concepts/compaction)总结你的对话之前，OpenClaw 会运行一个静默回合，提醒代理将重要的上下文保存到记忆文件中。这是默认开启的——你不需要配置任何东西。

要将该家务回合保持在本地模型上，设置精确的记忆刷新模型覆盖：

```json
{
  "agents": {
    "defaults": {
      "compaction": {
        "memoryFlush": {
          "model": "ollama/qwen3:8b"
        }
      }
    }
  }
}
```

覆盖仅应用于记忆刷新回合，不继承活动会话回退链。

<Tip>
记忆刷新防止压缩期间的上下文丢失。如果你的代理在对话中有尚未写入文件的重要事实，它们将在总结发生之前自动保存。
</Tip>

## 梦境

梦境是记忆的可选后台整合过程。它收集短期信号、评分候选者，并仅将合格的项目提升为长期记忆（`MEMORY.md`）。

它旨在保持长期记忆的高信号：

- **可选**：默认禁用。
- **定时**：启用时，`memory-core` 自动管理一个用于完整梦境扫描的重复定时任务。
- **阈值**：提升必须通过评分、召回频率和查询多样性门控。
- **可审查**：阶段摘要和日记条目写入 `DREAMS.md` 供人类审查。

有关阶段行为、评分信号和梦境日记的详细信息，参见[梦境](/zh-CN/concepts/dreaming)。

## 接地回填和实时提升

梦境系统现在有两个密切相关的审查通道：

- **实时梦境** 从短期梦境存储 `memory/.dreams/` 工作，是正常深度阶段用于决定什么可以升入 `MEMORY.md` 的内容。
- **接地回填** 读取历史 `memory/YYYY-MM-DD.md` 笔记作为独立的日期文件，并将结构化的审查输出写入 `DREAMS.md`。

当你想重放旧笔记并检查系统认为什么是持久的，而无需手动编辑 `MEMORY.md` 时，接地回填非常有用。

当你使用：

```bash
openclaw memory rem-backfill --path ./memory --stage-short-term
```

接地的持久候选不会被直接提升。它们被暂存到正常深度阶段已经使用的相同的短期梦境存储中。这意味着：

- `DREAMS.md` 保持为人类审查界面。
- 短期存储保持为机器面向的排序界面。
- `MEMORY.md` 仍然仅由深度提升写入。

如果你认为重放没有用，你可以删除暂存的工件，而无需触及普通日记条目或正常召回状态：

```bash
openclaw memory rem-backfill --rollback
openclaw memory rem-backfill --rollback-short-term
```

## CLI

```bash
openclaw memory status          # 检查索引状态和提供商
openclaw memory search "query"  # 从命令行搜索
openclaw memory index --force   # 重建索引
```

## 进一步阅读

- [内置记忆引擎](/zh-CN/concepts/memory-builtin)：默认 SQLite 后端。
- [QMD 记忆引擎](/zh-CN/concepts/memory-qmd)：高级本地优先侧车。
- [Honcho 记忆](/zh-CN/concepts/memory-honcho)：AI 原生的跨会话记忆。
- [记忆 LanceDB](/zh-CN/plugins/memory-lancedb)：LanceDB 支持的插件，具有 OpenAI 兼容嵌入。
- [记忆 Wiki](/zh-CN/plugins/memory-wiki)：编译的知识存储库和 wiki 原生工具。
- [记忆搜索](/zh-CN/concepts/memory-search)：搜索管道、提供商和调优。
- [梦境](/zh-CN/concepts/dreaming)：从短期回收到长期记忆的后台提升。
- [记忆配置参考](/zh-CN/reference/memory-config)：所有配置旋钮。
- [压缩](/zh-CN/concepts/compaction)：压缩如何与记忆交互。

## 相关文档

- [活动记忆](/zh-CN/concepts/active-memory)
- [记忆搜索](/zh-CN/concepts/memory-search)
- [内置记忆引擎](/zh-CN/concepts/memory-builtin)
- [Honcho 记忆](/zh-CN/concepts/memory-honcho)
- [记忆 LanceDB](/zh-CN/plugins/memory-lancedb)
- [承诺](/zh-CN/concepts/commitments)
