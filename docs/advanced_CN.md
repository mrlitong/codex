<!-- 翻译信息
原文档: docs/advanced.md
上游提交: 1772bf63
最后同步: 2025-10-17
翻译状态: 已完成
-->

## 高级主题

## 追踪 / 详细日志

由于 Codex 是用 Rust 编写的,它遵循 `RUST_LOG` 环境变量来配置日志行为。

TUI 默认使用 `RUST_LOG=codex_core=info,codex_tui=info`,日志消息会写入 `~/.codex/log/codex-tui.log`,因此你可以在单独的终端中运行以下命令来实时监控日志消息:

```
tail -F ~/.codex/log/codex-tui.log
```

相比之下,非交互式模式 (`codex exec`) 默认使用 `RUST_LOG=error`,但消息会内联打印,因此无需监控单独的文件。

有关配置选项的更多信息,请参阅 Rust 关于 [`RUST_LOG`](https://docs.rs/env_logger/latest/env_logger/#enabling-logging) 的文档。

## 模型上下文协议 (MCP)

Codex CLI 和 IDE 扩展是一个 MCP 客户端,这意味着它可以配置为连接到 MCP 服务器。有关更多信息,请参阅 [`config 文档`](./config.md#connecting-to-mcp-servers)。

## 将 Codex 用作 MCP 服务器

Codex CLI 也可以通过 `codex mcp-server` 作为 MCP _服务器_ 运行。例如,你可以使用 `codex mcp-server` 将 Codex 作为工具提供给多代理框架,如 OpenAI [Agents SDK](https://platform.openai.com/docs/guides/agents)。使用 `codex mcp` 可以单独在配置中添加/列出/获取/删除 MCP 服务器启动器。

### Codex MCP 服务器快速开始

你可以使用 [模型上下文协议检查器](https://modelcontextprotocol.io/legacy/tools/inspector) 启动 Codex MCP 服务器:

```bash
npx @modelcontextprotocol/inspector codex mcp-server
```

发送 `tools/list` 请求,你会看到有两个可用的工具:

**`codex`** - 运行 Codex 会话。接受与 Codex Config 结构匹配的配置参数。`codex` 工具接受以下属性:

| 属性                    | 类型    | 描述                                                                                                      |
| ----------------------- | ------- | --------------------------------------------------------------------------------------------------------- |
| **`prompt`** (必需)     | string  | 启动 Codex 对话的初始用户提示词。                                                                         |
| `approval-policy`       | string  | 模型生成的 shell 命令的批准策略: `untrusted`、`on-failure`、`never`。                                    |
| `base-instructions`     | string  | 用于替代默认指令的指令集。                                                                                |
| `config`                | object  | 将覆盖 `$CODEX_HOME/config.toml` 中设置的单个[配置设置](https://github.com/openai/codex/blob/main/docs/config.md#config)。 |
| `cwd`                   | string  | 会话的工作目录。如果是相对路径,则相对于服务器进程的当前目录进行解析。                                    |
| `include-plan-tool`     | boolean | 是否在对话中包含计划工具。                                                                                |
| `model`                 | string  | 模型名称的可选覆盖 (例如 `o3`、`o4-mini`)。                                                               |
| `profile`               | string  | 用于指定默认选项的 `config.toml` 中的配置配置文件。                                                      |
| `sandbox`               | string  | 沙箱模式: `read-only`、`workspace-write` 或 `danger-full-access`。                                       |

**`codex-reply`** - 通过提供对话 ID 和提示词来继续 Codex 会话。`codex-reply` 工具接受以下属性:

| 属性                            | 类型   | 描述                                 |
| ------------------------------- | ------ | ------------------------------------ |
| **`prompt`** (必需)             | string | 继续 Codex 对话的下一个用户提示词。 |
| **`conversationId`** (必需)     | string | 要继续的对话的 ID。                 |

### 试用

> [!TIP]
> Codex 通常需要几分钟才能运行。为了适应这一点,请在 ⛭ Configuration 下将 MCP 检查器的 Request 和 Total 超时时间调整为 600000ms (10 分钟)。

使用 MCP 检查器和 `codex mcp-server` 构建一个简单的井字棋游戏,设置如下:

**approval-policy:** never

**prompt:** Implement a simple tic-tac-toe game with HTML, Javascript, and CSS. Write the game in a single file called index.html.

**sandbox:** workspace-write

点击 "Run Tool",你应该会看到 Codex MCP 服务器在构建游戏时发出的一系列事件。
