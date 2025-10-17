<!-- 翻译信息
原文档: getting-started.md
上游提交: a30e5e40ee440d7122e7340fae5a56e7d013bc49
最后同步: 2025-10-17
翻译状态: 已完成
-->

## 快速开始

### CLI 使用方法

| 命令               | 用途                       | 示例                            |
| ------------------ | -------------------------- | ------------------------------- |
| `codex`            | 交互式终端界面（TUI）      | `codex`                         |
| `codex "..."`      | 为交互式 TUI 提供初始提示词 | `codex "fix lint errors"`       |
| `codex exec "..."` | 非交互式"自动化模式"       | `codex exec "explain utils.ts"` |

关键标志：`--model/-m`、`--ask-for-approval/-a`。

### 恢复交互式会话

- 运行 `codex resume` 显示会话选择器界面
- 恢复最近的会话：`codex resume --last`
- 按 ID 恢复：`codex resume <SESSION_ID>`（您可以从 /status 或 `~/.codex/sessions/` 获取会话 ID）

示例：

```shell
# 打开最近会话的选择器
codex resume

# 恢复最近的会话
codex resume --last

# 按 ID 恢复特定会话
codex resume 7f9f9a2e-1b3c-4c7a-9b0e-123456789abc
```

### 使用提示词作为输入运行

您也可以使用提示词作为输入来运行 Codex CLI：

```shell
codex "explain this codebase to me"
```

```shell
codex --full-auto "create the fanciest todo-list app"
```

就是这样 - Codex 会搭建文件框架、在沙箱中运行、安装任何缺失的依赖项，并向您展示实时结果。批准更改后，它们将被提交到您的工作目录。

### 示例提示词

以下是一些可以直接复制粘贴的小例子。将引号中的文本替换为您自己的任务。

| ✨  | 您输入的内容                                                                    | 发生的事情                                                   |
| --- | ------------------------------------------------------------------------------- | ------------------------------------------------------------ |
| 1   | `codex "Refactor the Dashboard component to React Hooks"`                       | Codex 重写类组件、运行 `npm test` 并显示差异。                |
| 2   | `codex "Generate SQL migrations for adding a users table"`                      | 推断您的 ORM、创建迁移文件并在沙箱数据库中运行它们。         |
| 3   | `codex "Write unit tests for utils/date.ts"`                                    | 生成测试、执行它们并迭代直到通过。                           |
| 4   | `codex "Bulk-rename *.jpeg -> *.jpg with git mv"`                               | 安全地重命名文件并更新导入/引用。                            |
| 5   | `codex "Explain what this regex does: ^(?=.*[A-Z]).{8,}$"`                      | 输出逐步的人类可读解释。                                     |
| 6   | `codex "Carefully review this repo, and propose 3 high impact well-scoped PRs"` | 在当前代码库中建议有影响力的 PR。                            |
| 7   | `codex "Look for vulnerabilities and create a security review report"`          | 查找并解释安全漏洞。                                         |

### 使用 AGENTS.md 提供记忆功能

您可以使用 `AGENTS.md` 文件为 Codex 提供额外的指令和指导。Codex 会在以下位置查找 `AGENTS.md` 文件，并自顶向下合并它们：

1. `~/.codex/AGENTS.md` - 个人全局指导
2. 仓库根目录的 `AGENTS.md` - 共享项目说明
3. 当前工作目录中的 `AGENTS.md` - 子文件夹/功能特定说明

有关如何使用 AGENTS.md 的更多信息，请参阅 [AGENTS.md 官方文档](https://agents.md/)。

### 提示与快捷方式

#### 使用 `@` 进行文件搜索

输入 `@` 会在工作区根目录触发模糊文件名搜索。使用上/下键在结果中选择，按 Tab 或 Enter 键用选定的路径替换 `@`。您可以使用 Esc 键取消搜索。

#### 图片输入

直接将图片粘贴到编辑器中（Ctrl+V / Cmd+V）以将它们附加到您的提示词。您也可以通过 CLI 使用 `-i/--image`（逗号分隔）附加文件：

```bash
codex -i screenshot.png "Explain this error"
codex --image img1.png,img2.jpg "Summarize these diagrams"
```

#### 按 Esc–Esc 编辑之前的消息

当聊天编辑器为空时，按 Esc 键进入"回溯"模式。再次按 Esc 键打开对话记录预览，突出显示最后一条用户消息；重复按 Esc 键可跳转到更早的用户消息。按 Enter 键确认，Codex 将从该点分叉对话、相应地修剪可见的对话记录，并在编辑器中预填充所选的用户消息，以便您可以编辑并重新提交。

在对话记录预览中，页脚会在编辑活动时显示 `Esc edit prev` 提示。

#### Shell 补全

通过以下命令生成 shell 补全脚本：

```shell
codex completion bash
codex completion zsh
codex completion fish
```

#### `--cd`/`-C` 标志

有时在运行 Codex 之前 `cd` 到您希望 Codex 用作"工作根目录"的目录并不方便。幸运的是，`codex` 支持 `--cd` 选项，因此您可以指定任何您想要的文件夹。您可以通过在新会话开始时检查 TUI 中报告的 **workdir** 来确认 Codex 是否遵循了 `--cd`。
