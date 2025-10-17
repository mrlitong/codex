<!-- 翻译信息
原文档: docs/exec.md
上游提交: 1772bf63
最后同步: 2025-10-17
翻译状态: 已完成
-->

## 非交互式模式

使用 Codex 的非交互式模式来自动化常见的工作流程。

```shell
codex exec "count the total number of lines of code in this project"
```

在非交互式模式下,Codex 不会请求命令或编辑批准。默认情况下,它以 `read-only` 模式运行,因此无法编辑文件或运行需要网络访问的命令。

使用 `codex exec --full-auto` 允许文件编辑。使用 `codex exec --sandbox danger-full-access` 允许编辑和网络命令。

### 默认输出模式

默认情况下,Codex 将其活动流式传输到 stderr,仅将代理的最终消息写入 stdout。这使得将 `codex exec` 通过管道传递到另一个工具时无需额外过滤。

要将 `codex exec` 的输出写入文件,除了使用 shell 重定向(如 `>`)之外,还有一个专用标志来指定输出文件: `-o`/`--output-last-message`。

### JSON 输出模式

`codex exec` 支持 `--json` 模式,在代理运行时将事件以 JSON Lines (JSONL) 格式流式传输到 stdout。

支持的事件类型:

- `thread.started` - 当线程启动或恢复时。
- `turn.started` - 当回合开始时。一个回合包含用户消息和助手响应之间的所有事件。
- `turn.completed` - 当回合完成时;包含 token 使用情况。
- `turn.failed` - 当回合失败时;包含错误详情。
- `item.started`/`item.updated`/`item.completed` - 当线程项被添加/更新/完成时。

支持的项目类型:

- `agent_message` - 助手消息。
- `reasoning` - 助手思考过程的摘要。
- `command_execution` - 助手执行命令。
- `file_change` - 助手进行文件更改。
- `mcp_tool_call` - 助手调用 MCP 工具。
- `web_search` - 助手执行网络搜索。

通常,`agent_message` 会在回合结束时添加。

示例输出:

```jsonl
{"type":"thread.started","thread_id":"0199a213-81c0-7800-8aa1-bbab2a035a53"}
{"type":"turn.started"}
{"type":"item.completed","item":{"id":"item_0","type":"reasoning","text":"**Searching for README files**"}}
{"type":"item.started","item":{"id":"item_1","type":"command_execution","command":"bash -lc ls","aggregated_output":"","status":"in_progress"}}
{"type":"item.completed","item":{"id":"item_1","type":"command_execution","command":"bash -lc ls","aggregated_output":"2025-09-11\nAGENTS.md\nCHANGELOG.md\ncliff.toml\ncodex-cli\ncodex-rs\ndocs\nexamples\nflake.lock\nflake.nix\nLICENSE\nnode_modules\nNOTICE\npackage.json\npnpm-lock.yaml\npnpm-workspace.yaml\nPNPM.md\nREADME.md\nscripts\nsdk\ntmp\n","exit_code":0,"status":"completed"}}
{"type":"item.completed","item":{"id":"item_2","type":"reasoning","text":"**Checking repository root for README**"}}
{"type":"item.completed","item":{"id":"item_3","type":"agent_message","text":"Yep — there's a `README.md` in the repository root."}}
{"type":"turn.completed","usage":{"input_tokens":24763,"cached_input_tokens":24448,"output_tokens":122}}
```

### 结构化输出

默认情况下,代理使用自然语言响应。使用 `--output-schema` 提供定义预期 JSON 输出的 JSON Schema。

JSON Schema 必须遵循[严格架构规则](https://platform.openai.com/docs/guides/structured-outputs)。

示例架构:

```json
{
  "type": "object",
  "properties": {
    "project_name": { "type": "string" },
    "programming_languages": { "type": "array", "items": { "type": "string" } }
  },
  "required": ["project_name", "programming_languages"],
  "additionalProperties": false
}
```

```shell
codex exec "Extract details of the project" --output-schema ~/schema.json
...

{"project_name":"Codex CLI","programming_languages":["Rust","TypeScript","Shell"]}
```

将 `--output-schema` 与 `-o` 结合使用,可以仅打印最终的 JSON 输出。你也可以将文件路径传递给 `-o` 以将 JSON 输出保存到文件。

### Git 仓库要求

Codex 需要 Git 仓库以避免破坏性更改。要禁用此检查,请使用 `codex exec --skip-git-repo-check`。

### 恢复非交互式会话

使用 `codex exec resume <SESSION_ID>` 或 `codex exec resume --last` 恢复之前的非交互式会话。这会保留对话上下文,以便你可以提出后续问题或向代理分配新任务。

```shell
codex exec "Review the change, look for use-after-free issues"
codex exec resume --last "Fix use-after-free issues"
```

仅保留对话上下文;你仍必须提供标志来自定义 Codex 行为。

```shell
codex exec --model gpt-5-codex --json "Review the change, look for use-after-free issues"
codex exec --model gpt-5 --json resume --last "Fix use-after-free issues"
```

## 认证

默认情况下,`codex exec` 将使用与 Codex CLI 和 VSCode 扩展相同的认证方法。你可以通过设置 `CODEX_API_KEY` 环境变量来覆盖 API 密钥。

```shell
CODEX_API_KEY=your-api-key-here codex exec "Fix merge conflict"
```

注意: `CODEX_API_KEY` 仅在 `codex exec` 中支持。
