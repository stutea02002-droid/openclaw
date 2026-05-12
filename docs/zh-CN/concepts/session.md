---
summary: "OpenClaw 如何管理会话会话"
read_when:
  - 你想了解会话路由和隔离时
  - 你想为多用户设置配置 DM 范围时
  - 你在调试每日或空闲会话重置时
title: "会话管理"
---

OpenClaw 将会话组织为**会话（sessions）**。每条消息根据其来源（私聊 DM、群聊、定时任务等）路由到一个会话。

## 消息如何路由

| 来源 | 行为 |
| --- | --- |
| 私聊 DM | 默认共享会话 |
| 群聊 | 每个群隔离 |
| 房间/频道 | 每个房间隔离 |
| 定时任务 | 每次运行新建会话 |
| Webhooks | 每个 hook 隔离 |

## DM 隔离

默认情况下，所有 DM 共享一个会话以保持连续性。这对于单用户设置来说没问题。

<Warning>
如果多人可以给代理发消息，请启用 DM 隔离。否则所有用户共享同一个对话上下文——Alice 的私聊消息对 Bob 可见。
</Warning>

**解决方法：**

```json5
{
  session: {
    dmScope: "per-channel-peer", // 按通道+发送者隔离
  },
}
```

其他选项：

- `main`（默认）—— 所有 DM 共享一个会话。
- `per-peer` —— 按发送者隔离（跨通道）。
- `per-channel-peer` —— 按通道 + 发送者隔离（推荐）。
- `per-account-channel-peer` —— 按账号 + 通道 + 发送者隔离。

<Tip>
如果同一个人从多个通道联系你，使用 `session.identityLinks` 链接他们的身份，使他们共享一个会话。
</Tip>

### 对接已链接的通道

对接命令允许用户将当前直接聊天会话的回复路由移动到另一个已链接的通道，而无需开始新会话。示例、配置和故障排除参见[通道对接](/zh-CN/concepts/channel-docking)。

使用 `openclaw security audit` 验证你的设置。

## 会话生命周期

会话会被重用，直到过期：

- **每日重置**（默认）—— 每天凌晨 4:00（Gateway 主机本地时间）新建会话。每日刷新基于当前 `sessionId` 的启动时间，而不是基于后续的元数据写入。
- **空闲重置**（可选）—— 在一段时间不活动后新建会话。设置 `session.reset.idleMinutes`。空闲刷新基于最后一次真实用户/通道交互，因此心跳、定时任务和执行系统事件不会让会话保持活跃。
- **手动重置** —— 在聊天中输入 `/new` 或 `/reset`。`/new <model>` 还会切换模型。

当每日重置和空闲重置都配置时，以先到者为准。心跳、定时任务、执行和其他系统事件回合可能会写入会话元数据，但这些写入不会延长每日或空闲重置的刷新时间。当重置滚动会话时，旧会话的排队系统事件通知会被丢弃，因此过时的后台更新不会被添加到新会话的第一个提示中。

具有活跃提供商拥有的 CLI 会话的会话不会被隐式每日默认值切断。当这些会话应该按计时器过期时，使用 `/reset` 或显式配置 `session.reset`。

## 状态存储在哪里

所有会话状态由 **Gateway** 拥有。UI 客户端向 Gateway 查询会话数据。

- **存储：** `~/.openclaw/agents/<agentId>/sessions/sessions.json`
- **转录：** `~/.openclaw/agents/<agentId>/sessions/<sessionId>.jsonl`

`sessions.json` 保留独立的生命周期时间戳：

- `sessionStartedAt`：当前 `sessionId` 何时启动；每日重置使用此字段。
- `lastInteractionAt`：最后一次用户/通道交互，延长空闲生命周期。
- `updatedAt`：最后一次存储行变更；用于列出和修剪，但对每日/空闲重置刷新不具有权威性。

缺少 `sessionStartedAt` 的旧行在可用时会从转录 JSONL 会话头中解析。如果旧行也缺少 `lastInteractionAt`，空闲刷新会回退到该会话启动时间，而不是后续记账写入。

## 会话维护

OpenClaw 会自动限制会话存储随时间增长。默认情况下，它以 `warn` 模式运行（报告将被清理的内容）。将 `session.maintenance.mode` 设置为 `"enforce"` 以进行自动清理：

```json5
{
  session: {
    maintenance: {
      mode: "enforce",
      pruneAfter: "30d",
      maxEntries: 500,
    },
  },
}
```

对于生产级的 `maxEntries` 限制，Gateway 运行时写入使用一个小型高水位缓冲区，并分批清理回配置的容量。会话存储读取不会在 Gateway 启动期间修剪或限制条目。这避免了在每次启动或孤立的定时会话上运行完整的存储清理。`openclaw sessions cleanup --enforce` 立即应用容量限制。

维护会保留持久的外部对话指针，包括群会话和话题范围的聊天会话，同时仍然允许合成的定时任务、hook、心跳、ACP 和子代理条目老化退出。

如果你之前使用了直接消息隔离，后来又返回 `session.dmScope` 到 `main`，可以使用 `openclaw sessions cleanup --dry-run --fix-dm-scope` 预览旧的 peer-keyed DM 行。应用相同的标志会退休那些旧的直接 DM 行，并将它们的转录保留为已删除的归档。

使用 `openclaw sessions cleanup --dry-run` 预览。

## 检查会话

- `openclaw status` —— 会话存储路径和最近活动。
- `openclaw sessions --json` —— 所有会话（用 `--active <minutes>` 过滤）。
- 聊天中的 `/status` —— 上下文使用、模型和开关。
- `/context list` —— 系统提示中包含什么。

## 进一步阅读

- [会话修剪](/zh-CN/concepts/session-pruning) —— 修剪工具结果
- [压缩](/zh-CN/concepts/compaction) —— 总结长对话
- [会话工具](/zh-CN/concepts/session-tool) —— 跨会话工作的代理工具
- [会话管理深入](/zh-CN/reference/session-management-compaction) —— 存储模式、转录、发送策略、来源元数据和高级配置
- [多代理](/zh-CN/concepts/multi-agent) —— 跨代理的路由和会话隔离
- [后台任务](/zh-CN/automation/tasks) —— 分离工作如何创建带会话引用的任务记录
- [通道路由](/zh-CN/channels/channel-routing) —— 入站消息如何路由到会话

## 相关文档

- [会话修剪](/zh-CN/concepts/session-pruning)
- [会话工具](/zh-CN/concepts/session-tool)
- [命令队列](/zh-CN/concepts/queue)
