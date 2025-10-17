<!-- 翻译信息
原文档: config.md
上游提交: 40fba1bb4c70a98138e54930e40d347055bc7b86
最后同步: 2025-10-17
翻译状态: 已完成
-->

# 配置

Codex 支持多种设置配置值的机制：

- 特定于配置的命令行标志，例如 `--model o3`（最高优先级）。
- 通用的 `-c`/`--config` 标志，接受 `key=value` 键值对，例如 `--config model="o3"`。
  - 键可以包含点号以设置比根级别更深的值，例如 `--config model_providers.openai.wire_api="chat"`。
  - 为了与 `config.toml` 保持一致，值是 TOML 格式的字符串而不是 JSON 格式，因此使用 `key='{a = 1, b = 2}'` 而不是 `key='{"a": 1, "b": 2}'`。
    - 值周围的引号是必需的，因为没有引号，您的 shell 会在空格处分割配置参数，导致 `codex` 接收到 `-c key={a` 以及（无效的）额外参数 `=`、`1,`、`b`、`=`、`2}`。
  - 值可以包含任何 TOML 对象，例如 `--config shell_environment_policy.include_only='["PATH", "HOME", "USER"]'`。
  - 如果 `value` 无法解析为有效的 TOML 值，则将其视为字符串值。这意味着 `-c model='"o3"'` 和 `-c model=o3` 是等效的。
    - 在第一种情况下，值是 TOML 字符串 `"o3"`，而在第二种情况下，值是 `o3`，这不是有效的 TOML，因此被视为 TOML 字符串 `"o3"`。
    - 因为引号由 shell 解释，`-c key="true"` 将在 TOML 中正确解释为 `key = true`（布尔值）而不是 `key = "true"`（字符串）。如果出于某种原因您需要字符串 `"true"`，则需要使用 `-c key='"true"'`（注意两组引号）。
- `$CODEX_HOME/config.toml` 配置文件，其中 `CODEX_HOME` 环境变量默认为 `~/.codex`。（注意 `CODEX_HOME` 也是日志和其他 Codex 相关信息存储的位置。）

`--config` 标志和 `config.toml` 文件都支持以下选项：

## model

Codex 应使用的模型。

```toml
model = "o3"  # 覆盖默认值 "gpt-5-codex"
```

## model_providers

此选项允许您覆盖和修改 Codex 捆绑的默认模型提供商集。此值是一个映射，其中键是与 `model_provider` 一起使用的值，用于选择相应的提供商。

例如，如果您想添加一个通过 chat completions API 使用 OpenAI 4o 模型的提供商，则可以添加以下配置：

```toml
# 回顾一下，在 TOML 中，根键必须列在表之前
model = "gpt-4o"
model_provider = "openai-chat-completions"

[model_providers.openai-chat-completions]
# 将在 Codex UI 中显示的提供商名称
name = "OpenAI using Chat Completions"
# 路径 `/chat/completions` 将附加到此 URL 以进行 chat completions 的 POST 请求
base_url = "https://api.openai.com/v1"
# 如果设置了 `env_key`，标识在使用此提供商的 Codex 时必须设置的环境变量。
# 环境变量的值必须非空，并将用于 POST 请求的 `Bearer TOKEN` HTTP 头
env_key = "OPENAI_API_KEY"
# wire_api 的有效值为 "chat" 和 "responses"。如果省略，默认为 "chat"
wire_api = "chat"
# 如有必要，需要添加到 URL 的额外查询参数
# 请参阅下面的 Azure 示例
query_params = {}
```

请注意，这使得可以将 Codex CLI 与非 OpenAI 模型一起使用，只要它们使用与 OpenAI chat completions API 兼容的 wire API。例如，您可以定义以下提供商以将 Codex CLI 与本地运行的 Ollama 一起使用：

```toml
[model_providers.ollama]
name = "Ollama"
base_url = "http://localhost:11434/v1"
```

或第三方提供商（使用不同的环境变量作为 API 密钥）：

```toml
[model_providers.mistral]
name = "Mistral"
base_url = "https://api.mistral.ai/v1"
env_key = "MISTRAL_API_KEY"
```

还可以配置提供商以在请求中包含额外的 HTTP 头。这些可以是硬编码值（`http_headers`）或从环境变量读取的值（`env_http_headers`）：

```toml
[model_providers.example]
# name, base_url, ...

# 这将向模型提供商的每个请求添加 HTTP 头 `X-Example-Header`，值为 `example-value`
http_headers = { "X-Example-Header" = "example-value" }

# 这将向模型提供商的每个请求添加 HTTP 头 `X-Example-Features`，
# 其值为 `EXAMPLE_FEATURES` 环境变量的值，_前提是_环境变量已设置且其值非空
env_http_headers = { "X-Example-Features" = "EXAMPLE_FEATURES" }
```

### Azure 模型提供商示例

请注意，Azure 要求将 `api-version` 作为查询参数传递，因此在定义 Azure 提供商时，请确保将其指定为 `query_params` 的一部分：

```toml
[model_providers.azure]
name = "Azure"
# 确保为此 URL 设置适当的子域
base_url = "https://YOUR_PROJECT_NAME.openai.azure.com/openai"
env_key = "AZURE_OPENAI_API_KEY"  # 或 "OPENAI_API_KEY"，无论您使用哪个
query_params = { api-version = "2025-04-01-preview" }
wire_api = "responses"
```

在启动 Codex 之前导出您的密钥：`export AZURE_OPENAI_API_KEY=…`

### 每个提供商的网络调优

以下可选设置控制**每个模型提供商**的重试行为和流式空闲超时。它们必须在 `config.toml` 中相应的 `[model_providers.<id>]` 块内指定。（较旧的版本接受顶级键；现在这些键被忽略。）

示例：

```toml
[model_providers.openai]
name = "OpenAI"
base_url = "https://api.openai.com/v1"
env_key = "OPENAI_API_KEY"
# 网络调优覆盖（全部可选；回退到内置默认值）
request_max_retries = 4            # 重试失败的 HTTP 请求
stream_max_retries = 10            # 重试断开的 SSE 流
stream_idle_timeout_ms = 300000    # 5分钟空闲超时
```

#### request_max_retries

Codex 将重试失败的 HTTP 请求到模型提供商的次数。默认为 `4`。

#### stream_max_retries

当流式响应中断时，Codex 将尝试重新连接的次数。默认为 `5`。

#### stream_idle_timeout_ms

Codex 在将连接视为丢失之前等待流式响应上的活动的时间。默认为 `300_000`（5 分钟）。

## model_provider

标识从 `model_providers` 映射中使用哪个提供商。默认为 `"openai"`。您可以通过 `OPENAI_BASE_URL` 环境变量覆盖内置 `openai` 提供商的 `base_url`。

请注意，如果您覆盖 `model_provider`，那么您可能也想覆盖 `model`。例如，如果您在本地运行带有 Mistral 的 ollama，那么除了 `model_providers` 映射中的新条目外，您还需要将以下内容添加到您的配置中：

```toml
model_provider = "ollama"
model = "mistral"
```

## approval_policy

确定何时应提示用户批准 Codex 是否可以执行命令：

```toml
# Codex 具有硬编码逻辑，定义了一组"受信任"的命令。
# 将 approval_policy 设置为 `untrusted` 意味着 Codex 将在运行不在"受信任"集中的命令之前提示用户。
#
# 有关允许最终用户定义自己的受信任命令的计划，请参阅 https://github.com/openai/codex/issues/1260
approval_policy = "untrusted"
```

如果您想在命令失败时收到通知，请使用 "on-failure"：

```toml
# 如果命令在沙箱中运行失败，Codex 会请求权限在沙箱外重试该命令
approval_policy = "on-failure"
```

如果您希望模型运行直到它决定需要请求您提供提升的权限，请使用 "on-request"：

```toml
# 模型决定何时升级
approval_policy = "on-request"
```

或者，您可以让模型运行直到完成，并且永远不要求运行具有提升权限的命令：

```toml
# 从不提示用户：如果命令失败，Codex 将自动尝试其他方法。
# 请注意，`exec` 子命令始终使用此模式。
approval_policy = "never"
```

## profiles

_配置文件_是可以一起设置的配置值集合。可以在 `config.toml` 中定义多个配置文件，您可以通过 `--profile` 标志在运行时指定要使用的配置文件。

以下是定义多个配置文件的 `config.toml` 示例：

```toml
model = "o3"
approval_policy = "untrusted"

# 设置 `profile` 等同于在命令行上指定 `--profile o3`，
# 尽管 `--profile` 标志仍可用于覆盖此值
profile = "o3"

[model_providers.openai-chat-completions]
name = "OpenAI using Chat Completions"
base_url = "https://api.openai.com/v1"
env_key = "OPENAI_API_KEY"
wire_api = "chat"

[profiles.o3]
model = "o3"
model_provider = "openai"
approval_policy = "never"
model_reasoning_effort = "high"
model_reasoning_summary = "detailed"

[profiles.gpt3]
model = "gpt-3.5-turbo"
model_provider = "openai-chat-completions"

[profiles.zdr]
model = "o3"
model_provider = "openai"
approval_policy = "on-failure"
```

用户可以在多个级别指定配置值。优先顺序如下：

1. 自定义命令行参数，例如 `--model o3`
2. 作为配置文件的一部分，其中 `--profile` 通过 CLI 指定（或在配置文件本身中）
3. 作为 `config.toml` 中的条目，例如 `model = "o3"`
4. Codex CLI 附带的默认值（即 Codex CLI 默认为 `gpt-5-codex`）

## model_reasoning_effort

如果所选模型已知支持推理（例如：`o3`、`o4-mini`、`codex-*`、`gpt-5`、`gpt-5-codex`），则在使用 Responses API 时默认启用推理。如 [OpenAI Platform 文档](https://platform.openai.com/docs/guides/reasoning?api-mode=responses#get-started-with-reasoning)中所述，可以设置为：

- `"minimal"`
- `"low"`
- `"medium"`（默认）
- `"high"`

注意：要最小化推理，请选择 `"minimal"`。

## model_reasoning_summary

如果模型名称以 `"o"` 开头（如 `"o3"` 或 `"o4-mini"`）或 `"codex"`，则在使用 Responses API 时默认启用推理。如 [OpenAI Platform 文档](https://platform.openai.com/docs/guides/reasoning?api-mode=responses#reasoning-summaries)中所述，可以设置为：

- `"auto"`（默认）
- `"concise"`
- `"detailed"`

要禁用推理摘要，请在配置中将 `model_reasoning_summary` 设置为 `"none"`：

```toml
model_reasoning_summary = "none"  # 禁用推理摘要
```

## model_verbosity

在使用 Responses API 时控制 GPT-5 系列模型的输出长度/详细程度。支持的值：

- `"low"`
- `"medium"`（省略时的默认值）
- `"high"`

设置后，Codex 会在请求有效负载中包含具有配置的详细程度的 `text` 对象，例如：`"text": { "verbosity": "low" }`。

示例：

```toml
model = "gpt-5"
model_verbosity = "low"
```

注意：这仅适用于使用 Responses API 的提供商。Chat Completions 提供商不受影响。

## model_supports_reasoning_summaries

默认情况下，仅在对已知支持它们的 OpenAI 模型的请求上设置 `reasoning`。要强制在对当前模型的请求上设置 `reasoning`，可以通过在 `config.toml` 中设置以下内容来强制执行此行为：

```toml
model_supports_reasoning_summaries = true
```

## sandbox_mode

Codex 在操作系统级别的沙箱内执行模型生成的 shell 命令。

在大多数情况下，您可以使用单个选项选择所需的行为：

```toml
# 与 `--sandbox read-only` 相同
sandbox_mode = "read-only"
```

默认策略是 `read-only`，这意味着命令可以读取磁盘上的任何文件，但尝试写入文件或访问网络将被阻止。

更宽松的策略是 `workspace-write`。指定后，Codex 任务的当前工作目录将可写（以及 macOS 上的 `$TMPDIR`）。请注意，CLI 默认使用生成它的目录作为 `cwd`，但可以使用 `--cwd/-C` 覆盖。

在 macOS 上（以及即将在 Linux 上），所有包含 `.git/` 文件夹_作为直接子级_的可写根（包括 `cwd`）将配置为 `.git/` 文件夹为只读，而 Git 仓库的其余部分将可写。这意味着默认情况下，像 `git commit` 这样的命令将失败（因为它需要写入 `.git/`），并且需要 Codex 请求权限。

```toml
# 与 `--sandbox workspace-write` 相同
sandbox_mode = "workspace-write"

# 仅在 `sandbox = "workspace-write"` 时适用的额外设置
[sandbox_workspace_write]
# 默认情况下，Codex 会话的 cwd 将可写，以及 $TMPDIR（如果设置）和 /tmp（如果存在）。
# 将相应的选项设置为 `true` 将覆盖这些默认值
exclude_tmpdir_env_var = false
exclude_slash_tmp = false

# 除 $TMPDIR 和 /tmp 之外的_额外_可写根的可选列表
writable_roots = ["/Users/YOU/.pyenv/shims"]

# 允许在沙箱内运行的命令发出出站网络请求。默认禁用
network_access = false
```

要完全禁用沙箱，请像这样指定 `danger-full-access`：

```toml
# 与 `--sandbox danger-full-access` 相同
sandbox_mode = "danger-full-access"
```

如果 Codex 在提供自己的沙箱的环境（例如 Docker 容器）中运行，使得进一步的沙箱是不必要的，则使用此选项是合理的。

但是，如果您尝试在不支持其原生沙箱机制的环境中使用 Codex，例如较旧的 Linux 内核或 Windows 上，则可能还需要使用此选项。

## 审批预设

Codex 提供三个主要的审批预设：

- Read Only（只读）：Codex 可以读取文件并回答问题；编辑、运行命令和网络访问需要批准。
- Auto（自动）：Codex 可以在工作区中读取文件、进行编辑和运行命令，无需批准；在工作区外或网络访问时需要批准。
- Full Access（完全访问）：完全磁盘和网络访问，无提示；极其危险。

您可以使用 `--ask-for-approval` 和 `--sandbox` 选项在命令行进一步自定义 Codex 的运行方式。

## 连接到 MCP 服务器

您可以配置 Codex 使用 [MCP 服务器](https://modelcontextprotocol.io/about)，以使 Codex 访问外部应用程序、资源或服务。

### 服务器配置

#### STDIO

[STDIO 服务器](https://modelcontextprotocol.io/specification/2025-06-18/basic/transports#stdio)是您可以直接通过计算机上的命令启动的 MCP 服务器。

```toml
# 顶级表名必须是 `mcp_servers`
# 子表名（本例中的 `server-name`）可以是您想要的任何名称
[mcp_servers.server_name]
command = "npx"
# 可选
args = ["-y", "mcp-server"]
# 可选：将额外的环境变量传播到 MCP 服务器
# 默认的环境变量白名单将传播到 MCP 服务器
# https://github.com/openai/codex/blob/main/codex-rs/rmcp-client/src/utils.rs#L82
env = { "API_KEY" = "value" }
# 或
[mcp_servers.server_name.env]
API_KEY = "value"
# 可选：将在 MCP 服务器环境中列入白名单的额外环境变量列表
env_vars = ["API_KEY2"]

# 可选：命令将从中运行的 cwd
cwd = "/Users/<user>/code/my-server"
```

#### Streamable HTTP

[Streamable HTTP 服务器](https://modelcontextprotocol.io/specification/2025-06-18/basic/transports#streamable-http)使 Codex 能够与通过 HTTP URL（在 localhost 或其他域上）访问的资源通信。

```toml
# Streamable HTTP 需要实验性的 rmcp 客户端
experimental_use_rmcp_client = true
[mcp_servers.figma]
url = "https://mcp.linear.app/mcp"
# 可选的包含 bearer token 的环境变量，用于身份验证
bearer_token_env_var = "<token>"
# 可选的具有硬编码值的标头映射
http_headers = { "HEADER_NAME" = "HEADER_VALUE" }
# 可选的标头映射，其值将替换为环境变量
env_http_headers = { "HEADER_NAME" = "ENV_VAR" }
```

对于 oauth 登录，您必须启用 `experimental_use_rmcp_client = true`，然后运行 `codex mcp login server_name`

### 其他配置选项

```toml
# 可选：覆盖默认的 10 秒启动超时
startup_timeout_sec = 20
# 可选：覆盖默认的 60 秒每工具超时
tool_timeout_sec = 30
# 可选：禁用服务器而不删除它
enabled = false
```

### 实验性 RMCP 客户端

Codex 正在过渡到[官方 Rust MCP SDK](https://github.com/modelcontextprotocol/rust-sdk)。

该标志启用了对 streamable HTTP 服务器的 OAuth 支持，并使用新的 STDIO 客户端实现。

请尝试并报告新客户端的问题。要启用它，请将其添加到 `config.toml` 的顶层

```toml
experimental_use_rmcp_client = true

[mcp_servers.server_name]
…
```

### MCP CLI 命令

```shell
# 列出所有可用命令
codex mcp --help

# 添加服务器（env 可以重复；`--` 分隔启动器命令）
codex mcp add docs -- docs-server --port 4000

# 列出配置的服务器（漂亮的表格或 JSON）
codex mcp list
codex mcp list --json

# 显示一个服务器（表格或 JSON）
codex mcp get docs
codex mcp get docs --json

# 删除服务器
codex mcp remove docs

# 登录支持 oauth 的 streamable HTTP 服务器
codex mcp login SERVER_NAME

# 从支持 oauth 的 streamable HTTP 服务器注销
codex mcp logout SERVER_NAME
```

## 有用的 MCP 示例

有一个不断增长的有用 MCP 服务器列表，在使用 Codex 时可能会有所帮助。

我们看到的一些最常见的 MCP 是：

- [Context7](https://github.com/upstash/context7) — 连接到各种最新的开发者文档
- Figma [本地](https://developers.figma.com/docs/figma-mcp-server/local-server-installation/)和[远程](https://developers.figma.com/docs/figma-mcp-server/remote-server-installation/) - 访问您的 Figma 设计
- [Playwright](https://www.npmjs.com/package/@playwright/mcp) - 使用 Playwright 控制和检查浏览器
- [Chrome Developer Tools](https://github.com/ChromeDevTools/chrome-devtools-mcp/) — 控制和检查 Chrome 浏览器
- [Sentry](https://docs.sentry.io/product/sentry-mcp/#codex) — 访问您的 Sentry 日志
- [GitHub](https://github.com/github/github-mcp-server) — 控制您的 GitHub 帐户，超出 git 允许的范围（如控制 PR、issue 等）

## shell_environment_policy

Codex 生成子进程（例如，当执行助手建议的 `local_shell` 工具调用时）。默认情况下，它现在将**您的完整环境**传递给这些子进程。您可以通过 `config.toml` 中的 **`shell_environment_policy`** 块调整此行为：

```toml
[shell_environment_policy]
# inherit 可以是 "all"（默认）、"core" 或 "none"
inherit = "core"
# 设置为 true 以*跳过*`"*KEY*"` 和 `"*TOKEN*"` 的过滤器
ignore_default_excludes = false
# 排除模式（不区分大小写的 glob）
exclude = ["AWS_*", "AZURE_*"]
# 强制设置/覆盖值
set = { CI = "1" }
# 如果提供，*仅*保留与这些模式匹配的变量
include_only = ["PATH", "HOME"]
```

| 字段                      | 类型                 | 默认值  | 描述                                                                                                                          |
| ------------------------- | -------------------- | ------- | ----------------------------------------------------------------------------------------------------------------------------- |
| `inherit`                 | string               | `all`   | 环境的起始模板：<br>`all`（克隆完整父环境）、`core`（`HOME`、`PATH`、`USER`、…）或 `none`（从空开始）。                      |
| `ignore_default_excludes` | boolean              | `false` | 当为 `false` 时，Codex 会在其他规则运行之前删除**名称**包含 `KEY`、`SECRET` 或 `TOKEN`（不区分大小写）的任何变量。          |
| `exclude`                 | array<string>        | `[]`    | 默认过滤器之后要删除的不区分大小写的 glob 模式。<br>示例：`"AWS_*"`、`"AZURE_*"`。                                           |
| `set`                     | table<string,string> | `{}`    | 显式键/值覆盖或添加 - 始终优先于继承的值。                                                                                   |
| `include_only`            | array<string>        | `[]`    | 如果非空，则为白名单模式；只有匹配_一个_模式的变量才能在最后一步中保留。（通常与 `inherit = "all"` 一起使用。）             |

模式是**glob 样式**，而不是完整的正则表达式：`*` 匹配任意数量的字符，`?` 恰好匹配一个，支持像 `[A-Z]`/`[^0-9]` 这样的字符类。匹配始终**不区分大小写**。此语法在代码中记录为 `EnvironmentVariablePattern`（请参阅 `core/src/config_types.rs`）。

如果您只需要一个包含几个自定义条目的干净状态，可以这样写：

```toml
[shell_environment_policy]
inherit = "none"
set = { PATH = "/usr/bin", MY_FLAG = "1" }
```

目前，假设网络被禁用，`CODEX_SANDBOX_NETWORK_DISABLED=1` 也会添加到环境中。这是不可配置的。

## otel

Codex 可以发出 [OpenTelemetry](https://opentelemetry.io/) **日志事件**，描述每次运行：出站 API 请求、流式响应、用户输入、工具批准决策以及每次工具调用的结果。默认情况下**禁用**导出，因此本地运行保持独立。通过添加 `[otel]` 表并选择导出器来选择启用。

```toml
[otel]
environment = "staging"   # 默认为 "dev"
exporter = "none"          # 默认为 "none"；设置为 otlp-http 或 otlp-grpc 以发送事件
log_user_prompt = false    # 默认为 false；除非明确启用，否则编辑提示文本
```

Codex 使用 `service.name = $ORIGINATOR`（在 `originator` 头中发送的相同值，默认为 `codex_cli_rs`）、CLI 版本和 `env` 属性标记每个导出的事件，以便下游收集器可以区分 dev/staging/prod 流量。只有在 `codex_otel` crate 内生成的遥测数据（下面列出的事件）才会转发到导出器。

### 事件目录

每个事件共享一组通用的元数据字段：`event.timestamp`、`conversation.id`、`app.version`、`auth_mode`（如果可用）、`user.account_id`（如果可用）、`user.email`（如果可用）、`terminal.type`、`model` 和 `slug`。

启用 OTEL 后，Codex 会发出以下事件类型（除上述元数据外）：

- `codex.conversation_starts`
  - `provider_name`
  - `reasoning_effort`（可选）
  - `reasoning_summary`
  - `context_window`（可选）
  - `max_output_tokens`（可选）
  - `auto_compact_token_limit`（可选）
  - `approval_policy`
  - `sandbox_policy`
  - `mcp_servers`（逗号分隔列表）
  - `active_profile`（可选）
- `codex.api_request`
  - `attempt`
  - `duration_ms`
  - `http.response.status_code`（可选）
  - `error.message`（失败）
- `codex.sse_event`
  - `event.kind`
  - `duration_ms`
  - `error.message`（失败）
  - `input_token_count`（仅响应）
  - `output_token_count`（仅响应）
  - `cached_token_count`（仅响应，可选）
  - `reasoning_token_count`（仅响应，可选）
  - `tool_token_count`（仅响应）
- `codex.user_prompt`
  - `prompt_length`
  - `prompt`（除非 `log_user_prompt = true`，否则编辑）
- `codex.tool_decision`
  - `tool_name`
  - `call_id`
  - `decision`（`approved`、`approved_for_session`、`denied` 或 `abort`）
  - `source`（`config` 或 `user`）
- `codex.tool_result`
  - `tool_name`
  - `call_id`（可选）
  - `arguments`（可选）
  - `duration_ms`（工具的执行时间）
  - `success`（`"true"` 或 `"false"`）
  - `output`

这些事件形状可能会随着我们的迭代而改变。

### 选择导出器

设置 `otel.exporter` 以控制事件的去向：

- `none` – 使检测保持活动状态但跳过导出。这是默认值。
- `otlp-http` – 将 OTLP 日志记录发布到 OTLP/HTTP 收集器。指定收集器期望的端点、协议和头：

  ```toml
  [otel]
  exporter = { otlp-http = {
    endpoint = "https://otel.example.com/v1/logs",
    protocol = "binary",
    headers = { "x-otlp-api-key" = "${OTLP_TOKEN}" }
  }}
  ```

- `otlp-grpc` – 通过 gRPC 流式传输 OTLP 日志记录。提供端点和任何元数据头：

  ```toml
  [otel]
  exporter = { otlp-grpc = {
    endpoint = "https://otel.example.com:4317",
    headers = { "x-otlp-meta" = "abc123" }
  }}
  ```

如果导出器是 `none`，则不会在任何地方写入任何内容；否则，您必须运行或指向您自己的收集器。所有导出器都在后台批处理工作器上运行，该工作器在关闭时刷新。

如果您从源代码构建 Codex，OTEL crate 仍然在 `otel` 功能标志后面；官方预构建的二进制文件在启用该功能的情况下发布。当该功能被禁用时，遥测挂钩将变为无操作，因此 CLI 继续运行而没有额外的依赖项。

## notify

指定一个将被执行以获取 Codex 生成的事件通知的程序。请注意，该程序将接收通知参数作为 JSON 字符串，例如：

```json
{
  "type": "agent-turn-complete",
  "thread-id": "b5f6c1c2-1111-2222-3333-444455556666",
  "turn-id": "12345",
  "input-messages": ["Rename `foo` to `bar` and update the callsites."],
  "last-assistant-message": "Rename complete and verified `cargo build` succeeds."
}
```

`"type"` 属性将始终被设置。目前，`"agent-turn-complete"` 是唯一支持的通知类型。

`"thread-id"` 包含一个标识生成通知的 Codex 会话的字符串；您可以使用它来关联属于同一任务的多个回合。

例如，这是一个 Python 脚本，它解析 JSON 并决定是否在 macOS 上使用 [terminal-notifier](https://github.com/julienXX/terminal-notifier) 显示桌面推送通知：

```python
#!/usr/bin/env python3

import json
import subprocess
import sys


def main() -> int:
    if len(sys.argv) != 2:
        print("Usage: notify.py <NOTIFICATION_JSON>")
        return 1

    try:
        notification = json.loads(sys.argv[1])
    except json.JSONDecodeError:
        return 1

    match notification_type := notification.get("type"):
        case "agent-turn-complete":
            assistant_message = notification.get("last-assistant-message")
            if assistant_message:
                title = f"Codex: {assistant_message}"
            else:
                title = "Codex: Turn Complete!"
            input_messages = notification.get("input-messages", [])
            message = " ".join(input_messages)
            title += message
        case _:
            print(f"not sending a push notification for: {notification_type}")
            return 0

    thread_id = notification.get("thread-id", "")

    subprocess.check_output(
        [
            "terminal-notifier",
            "-title",
            title,
            "-message",
            message,
            "-group",
            "codex-" + thread_id,
            "-ignoreDnD",
            "-activate",
            "com.googlecode.iterm2",
        ]
    )

    return 0


if __name__ == "__main__":
    sys.exit(main())
```

要让 Codex 使用此脚本进行通知，您可以通过 `~/.codex/config.toml` 中的 `notify` 进行配置，使用计算机上 `notify.py` 的适当路径：

```toml
notify = ["python3", "/Users/mbolin/.codex/notify.py"]
```

> [!NOTE]
> 使用 `notify` 进行自动化和集成：Codex 使用单个 JSON 参数为每个事件调用您的外部程序，独立于 TUI。如果您只想在使用 TUI 时使用轻量级桌面通知，请优先使用 `tui.notifications`，它使用终端转义代码，不需要外部程序。您可以同时启用两者；`tui.notifications` 涵盖 TUI 内警报（例如，批准提示），而 `notify` 最适合系统级挂钩或自定义通知程序。目前，`notify` 仅发出 `agent-turn-complete`，而 `tui.notifications` 支持 `agent-turn-complete` 和 `approval-requested`，并带有可选过滤。

## history

默认情况下，Codex CLI 将发送到模型的消息记录在 `$CODEX_HOME/history.jsonl` 中。请注意，在 UNIX 上，文件权限设置为 `o600`，因此它应该只能由所有者读取和写入。

要禁用此行为，请按如下方式配置 `[history]`：

```toml
[history]
persistence = "none"  # "save-all" 是默认值
```

## file_opener

标识用于超链接模型输出中引用的编辑器/URI 方案。如果设置，模型输出中对文件的引用将使用指定的 URI 方案进行超链接，以便可以从终端 ctrl/cmd 单击它们以打开它们。

例如，如果模型输出包含诸如 `【F:/home/user/project/main.py†L42-L50】` 之类的引用，则这将被重写为链接到 URI `vscode://file/home/user/project/main.py:42`。

请注意，这**不是**通用编辑器设置（如 `$EDITOR`），因为它只接受一组固定的值：

- `"vscode"`（默认）
- `"vscode-insiders"`
- `"windsurf"`
- `"cursor"`
- `"none"` 以明确禁用此功能

目前，`"vscode"` 是默认值，尽管 Codex 不会验证是否已安装 VS Code。因此，`file_opener` 将来可能默认为 `"none"` 或其他内容。

## hide_agent_reasoning

Codex 间歇性地发出"推理"事件，显示模型在产生最终答案之前的内部"思考"。一些用户可能会发现这些事件令人分心，特别是在 CI 日志或最小终端输出中。

将 `hide_agent_reasoning` 设置为 `true` 会在 **TUI** 以及无头 `exec` 子命令中抑制这些事件：

```toml
hide_agent_reasoning = true   # 默认为 false
```

## show_raw_agent_reasoning

在可用时显示模型的原始思维链（"原始推理内容"）。

注意：

- 仅在所选模型/提供商实际发出原始推理内容时才生效。许多模型不支持。当不支持时，此选项没有可见效果。
- 原始推理可能包括中间思想或敏感上下文。仅当可接受您的工作流程时才启用。

示例：

```toml
show_raw_agent_reasoning = true  # 默认为 false
```

## model_context_window

模型的上下文窗口大小，以 token 为单位。

通常，Codex 知道最常见 OpenAI 模型的上下文窗口，但如果您使用带有旧版本 Codex CLI 的新模型，则可以使用 `model_context_window` 告诉 Codex 在对话期间剩余多少上下文时要使用什么值。

## model_max_output_tokens

这类似于 `model_context_window`，但用于模型的最大输出 token 数。

## project_doc_max_bytes

从 `AGENTS.md` 文件读取的最大字节数，以包含在会话第一轮发送的指令中。默认为 32 KiB。

## project_doc_fallback_filenames

当给定目录级别缺少 `AGENTS.md` 时要查找的额外文件名的有序列表。CLI 始终首先检查 `AGENTS.md`；配置的回退按提供的顺序尝试。这让已经使用备用指令文件（例如 `CLAUDE.md`）的 monorepo 在您迁移到 `AGENTS.md` 时可以开箱即用。

```toml
project_doc_fallback_filenames = ["CLAUDE.md", ".exampleagentrules.md"]
```

我们建议将指令迁移到 AGENTS.md；其他文件名可能会降低模型性能。

## tui

特定于 TUI 的选项。

```toml
[tui]
# 在需要批准或完成回合时发送桌面通知
# 默认为 false
notifications = true

# 您可以选择过滤到特定的通知类型
# 可用类型为 "agent-turn-complete" 和 "approval-requested"
notifications = [ "agent-turn-complete", "approval-requested" ]
```

> [!NOTE]
> Codex 使用终端转义代码发出桌面通知。并非所有终端都支持这些（值得注意的是，macOS Terminal.app 和 VS Code 的终端不支持自定义通知。iTerm2、Ghostty 和 WezTerm 支持这些通知）。

> [!NOTE]
> `tui.notifications` 是内置的，仅限于 TUI 会话。对于程序化或跨环境通知 - 或与特定于操作系统的通知程序集成 - 使用顶级 `notify` 选项运行接收事件 JSON 的外部程序。这两个设置是独立的，可以一起使用。

## 配置参考

| 键                                               | 类型/值                                                       | 注释                                                                                                               |
| ------------------------------------------------ | ------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| `model`                                          | string                                                        | 要使用的模型（例如，`gpt-5-codex`）。                                                                             |
| `model_provider`                                 | string                                                        | 来自 `model_providers` 的提供商 ID（默认：`openai`）。                                                            |
| `model_context_window`                           | number                                                        | 上下文窗口 token。                                                                                                 |
| `model_max_output_tokens`                        | number                                                        | 最大输出 token。                                                                                                   |
| `approval_policy`                                | `untrusted` \| `on-failure` \| `on-request` \| `never`        | 何时提示批准。                                                                                                     |
| `sandbox_mode`                                   | `read-only` \| `workspace-write` \| `danger-full-access`      | 操作系统沙箱策略。                                                                                                 |
| `sandbox_workspace_write.writable_roots`         | array<string>                                                 | workspace-write 中的额外可写根。                                                                                   |
| `sandbox_workspace_write.network_access`         | boolean                                                       | 在 workspace-write 中允许网络（默认：false）。                                                                    |
| `sandbox_workspace_write.exclude_tmpdir_env_var` | boolean                                                       | 从可写根中排除 `$TMPDIR`（默认：false）。                                                                         |
| `sandbox_workspace_write.exclude_slash_tmp`      | boolean                                                       | 从可写根中排除 `/tmp`（默认：false）。                                                                            |
| `disable_response_storage`                       | boolean                                                       | ZDR 组织所需。                                                                                                     |
| `notify`                                         | array<string>                                                 | 用于通知的外部程序。                                                                                               |
| `instructions`                                   | string                                                        | 当前被忽略；使用 `experimental_instructions_file` 或 `AGENTS.md`。                                                |
| `mcp_servers.<id>.command`                       | string                                                        | MCP 服务器启动器命令（仅 stdio 服务器）。                                                                         |
| `mcp_servers.<id>.args`                          | array<string>                                                 | MCP 服务器参数（仅 stdio 服务器）。                                                                               |
| `mcp_servers.<id>.env`                           | map<string,string>                                            | MCP 服务器环境变量（仅 stdio 服务器）。                                                                           |
| `mcp_servers.<id>.url`                           | string                                                        | MCP 服务器 URL（仅 streamable http 服务器）。                                                                     |
| `mcp_servers.<id>.bearer_token_env_var`          | string                                                        | 包含 bearer token 的环境变量，用于身份验证（仅 streamable http 服务器）。                                         |
| `mcp_servers.<id>.enabled`                       | boolean                                                       | 当为 false 时，Codex 跳过启动服务器（默认：true）。                                                               |
| `mcp_servers.<id>.startup_timeout_sec`           | number                                                        | 启动超时（秒）（默认：10）。超时适用于初始化 MCP 服务器和初始列出工具。                                           |
| `mcp_servers.<id>.tool_timeout_sec`              | number                                                        | 每工具超时（秒）（默认：60）。接受小数值；省略以使用默认值。                                                      |
| `model_providers.<id>.name`                      | string                                                        | 显示名称。                                                                                                         |
| `model_providers.<id>.base_url`                  | string                                                        | API 基础 URL。                                                                                                     |
| `model_providers.<id>.env_key`                   | string                                                        | API 密钥的环境变量。                                                                                               |
| `model_providers.<id>.wire_api`                  | `chat` \| `responses`                                         | 使用的协议（默认：`chat`）。                                                                                       |
| `model_providers.<id>.query_params`              | map<string,string>                                            | 额外的查询参数（例如，Azure `api-version`）。                                                                     |
| `model_providers.<id>.http_headers`              | map<string,string>                                            | 额外的静态头。                                                                                                     |
| `model_providers.<id>.env_http_headers`          | map<string,string>                                            | 来自环境变量的头。                                                                                                 |
| `model_providers.<id>.request_max_retries`       | number                                                        | 每个提供商的 HTTP 重试次数（默认：4）。                                                                           |
| `model_providers.<id>.stream_max_retries`        | number                                                        | SSE 流重试次数（默认：5）。                                                                                       |
| `model_providers.<id>.stream_idle_timeout_ms`    | number                                                        | SSE 空闲超时（ms）（默认：300000）。                                                                              |
| `project_doc_max_bytes`                          | number                                                        | 从 `AGENTS.md` 读取的最大字节数。                                                                                  |
| `profile`                                        | string                                                        | 活动配置文件名称。                                                                                                 |
| `profiles.<name>.*`                              | various                                                       | 相同键的配置文件作用域覆盖。                                                                                       |
| `history.persistence`                            | `save-all` \| `none`                                          | 历史文件持久性（默认：`save-all`）。                                                                              |
| `history.max_bytes`                              | number                                                        | 当前被忽略（未强制执行）。                                                                                         |
| `file_opener`                                    | `vscode` \| `vscode-insiders` \| `windsurf` \| `cursor` \| `none` | 可点击引用的 URI 方案（默认：`vscode`）。                                                                         |
| `tui`                                            | table                                                         | TUI 特定选项。                                                                                                     |
| `tui.notifications`                              | boolean \| array<string>                                      | 在 TUI 中启用桌面通知（默认：false）。                                                                            |
| `hide_agent_reasoning`                           | boolean                                                       | 隐藏模型推理事件。                                                                                                 |
| `show_raw_agent_reasoning`                       | boolean                                                       | 显示原始推理（如果可用）。                                                                                         |
| `model_reasoning_effort`                         | `minimal` \| `low` \| `medium` \| `high`                      | Responses API 推理努力程度。                                                                                       |
| `model_reasoning_summary`                        | `auto` \| `concise` \| `detailed` \| `none`                   | 推理摘要。                                                                                                         |
| `model_verbosity`                                | `low` \| `medium` \| `high`                                   | GPT-5 文本详细程度（Responses API）。                                                                              |
| `model_supports_reasoning_summaries`             | boolean                                                       | 强制启用推理摘要。                                                                                                 |
| `model_reasoning_summary_format`                 | `none` \| `experimental`                                      | 强制推理摘要格式。                                                                                                 |
| `chatgpt_base_url`                               | string                                                        | ChatGPT 身份验证流程的基础 URL。                                                                                   |
| `experimental_resume`                            | string (path)                                                 | 恢复 JSONL 路径（内部/实验性）。                                                                                   |
| `experimental_instructions_file`                 | string (path)                                                 | 替换内置指令（实验性）。                                                                                           |
| `experimental_use_exec_command_tool`             | boolean                                                       | 使用实验性 exec 命令工具。                                                                                         |
| `responses_originator_header_internal_override`  | string                                                        | 覆盖 `originator` 头值。                                                                                           |
| `projects.<path>.trust_level`                    | string                                                        | 将项目/工作树标记为受信任（仅识别 `"trusted"`）。                                                                 |
| `tools.web_search`                               | boolean                                                       | 启用 web 搜索工具（别名：`web_search_request`）（默认：false）。                                                  |
