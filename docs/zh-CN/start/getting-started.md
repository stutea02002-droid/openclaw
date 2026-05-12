---
summary: "几分钟内完成 OpenClaw 安装并运行你的第一次聊天。"
read_when:
  - 首次从零开始设置
  - 你想最快获得一个可用的聊天环境
title: "开始使用"
---

安装 OpenClaw、运行引导向导，与你的 AI 助手聊天——大约只需 5 分钟。完成后你将拥有一个运行中的 Gateway、已配置的认证，以及一个可用的聊天会话。

## 你需要什么

- **Node.js** — 推荐 Node 24（也支持 Node 22.16+）
- **API 密钥** — 来自模型提供商（Anthropic、OpenAI、Google 等）——引导向导会提示你

<Tip>
用 `node --version` 检查你的 Node 版本。
**Windows 用户：** 原生 Windows 和 WSL2 都受支持。WSL2 更稳定，推荐获得完整体验。查看 [Windows](/zh-CN/platforms/windows)。
需要安装 Node？查看 [Node 设置](/zh-CN/install/node)。
</Tip>

## 快速设置

<Steps>
  <Step title="安装 OpenClaw">
    <Tabs>
      <Tab title="macOS / Linux">
        ```bash
        curl -fsSL https://openclaw.ai/install.sh | bash
        ```
        <img
  src="/assets/install-script.svg"
  alt="安装脚本流程"
  className="rounded-lg"
/>
      </Tab>
      <Tab title="Windows (PowerShell)">
        ```powershell
        iwr -useb https://openclaw.ai/install.ps1 | iex
        ```
      </Tab>
    </Tabs>

    <Note>
    其他安装方法（Docker、Nix、npm）：[安装](/zh-CN/install)。
    </Note>

  </Step>
  <Step title="运行引导向导">
    ```bash
    openclaw onboard --install-daemon
    ```

    向导会引导你选择模型提供商、设置 API 密钥，以及配置 Gateway。大约需要 2 分钟。

    完整参考见[引导向导（CLI）](/zh-CN/start/wizard)。

  </Step>
  <Step title="验证 Gateway 正在运行">
    ```bash
    openclaw gateway status
    ```

    你应该能看到 Gateway 正在监听 18789 端口。

  </Step>
  <Step title="打开控制面板">
    ```bash
    openclaw dashboard
    ```

    这会在浏览器中打开控制面板。如果能正常加载，说明一切工作正常。

  </Step>
  <Step title="发送你的第一条消息">
    在控制面板聊天中输入消息，你应该会收到 AI 回复。

    想从手机聊天？最快设置的通道是
    [Telegram](/zh-CN/channels/telegram)（只需要一个机器人令牌）。查看[通道](/zh-CN/channels)了解所有选项。

  </Step>
</Steps>

<Accordion title="高级：挂载自定义控制面板构建">
  如果你维护了一个本地化或定制化的仪表板构建，将
  `gateway.controlUi.root` 指向包含你构建的静态资源和 `index.html` 的目录。

```bash
mkdir -p "$HOME/.openclaw/control-ui-custom"
# 将你构建的静态文件复制到该目录。
```

然后设置：

```json
{
  "gateway": {
    "controlUi": {
      "enabled": true,
      "root": "$HOME/.openclaw/control-ui-custom"
    }
  }
}
```

重启 Gateway 并重新打开控制面板：

```bash
openclaw gateway restart
openclaw dashboard
```

</Accordion>

## 接下来做什么

<Columns>
  <Card title="连接通道" href="/zh-CN/channels" icon="message-square">
    Discord、飞书、iMessage、Matrix、Microsoft Teams、Signal、Slack、Telegram、WhatsApp、Zalo 等。
  </Card>
  <Card title="配对与安全" href="/zh-CN/channels/pairing" icon="shield">
    控制谁可以向你的代理发消息。
  </Card>
  <Card title="配置 Gateway" href="/zh-CN/gateway/configuration" icon="settings">
    模型、工具、沙箱和高级设置。
  </Card>
  <Card title="浏览工具" href="/zh-CN/tools" icon="wrench">
    浏览器、执行、网页搜索、技能和插件。
  </Card>
</Columns>

<Accordion title="高级：环境变量">
  如果你以服务账户运行 OpenClaw 或需要自定义路径：

- `OPENCLAW_HOME` — 内部路径解析的主目录
- `OPENCLAW_STATE_DIR` — 覆盖状态目录
- `OPENCLAW_CONFIG_PATH` — 覆盖配置文件路径

完整参考：[环境变量](/zh-CN/help/environment)。
</Accordion>

## 相关文档

- [安装概览](/zh-CN/install)
- [通道概览](/zh-CN/channels)
- [设置](/zh-CN/start/setup)
