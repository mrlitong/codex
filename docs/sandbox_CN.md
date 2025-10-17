<!-- 翻译信息
原文档: sandbox.md
上游提交: 77a8b7fdeb4f9eff816acc14d0e96d3d92e61210
最后同步: 2025-10-17
翻译状态: 已完成
-->

## 沙箱与权限审批

### 审批模式

我们为 Codex 在您计算机上的工作方式选择了一个强大的默认设置：`Auto`（自动）。在此审批模式下，Codex 可以自动读取文件、进行编辑并在工作目录中运行命令。但是，Codex 需要您的批准才能在工作目录之外工作或访问网络。

当您只想聊天，或者想要在深入之前进行规划时，可以使用 `/approvals` 命令切换到 `Read Only`（只读）模式。

如果您需要 Codex 读取文件、进行编辑并运行具有网络访问权限的命令，而无需批准，可以使用 `Full Access`（完全访问）。这样做前请谨慎行事。

#### 默认设置和建议

- Codex 默认在沙箱中运行，具有强大的防护：它阻止编辑工作区外的文件，并阻止网络访问（除非启用）。
- 启动时，Codex 会检测文件夹是否受版本控制，并建议：
  - 受版本控制的文件夹：`Auto`（工作区写入 + 按需批准）
  - 非版本控制的文件夹：`Read Only`（只读）
- 工作区包括当前目录和临时目录（如 `/tmp`）。使用 `/status` 命令查看工作区中包含哪些目录。
- 您可以显式设置这些：
  - `codex --sandbox workspace-write --ask-for-approval on-request`
  - `codex --sandbox read-only --ask-for-approval on-request`

### 可以在没有任何审批的情况下运行吗？

是的，您可以使用 `--ask-for-approval never` 禁用所有审批提示。此选项适用于所有 `--sandbox` 模式，因此您仍然可以完全控制 Codex 的自主程度。它将在您提供的任何约束下尽最大努力。

### 常见的沙箱 + 审批组合

| 意图                         | 标志                                                                                        | 效果                                                                                                             |
| ---------------------------- | ------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| 安全的只读浏览               | `--sandbox read-only --ask-for-approval on-request`                                         | Codex 可以读取文件并回答问题。Codex 需要批准才能进行编辑、运行命令或访问网络。                                   |
| 只读非交互式（CI）           | `--sandbox read-only --ask-for-approval never`                                              | 仅读取；从不升级                                                                                                 |
| 允许编辑仓库，有风险时询问   | `--sandbox workspace-write --ask-for-approval on-request`                                   | Codex 可以读取文件、进行编辑并在工作区中运行命令。Codex 需要批准才能在工作区外执行操作或访问网络。               |
| Auto（预设）                 | `--full-auto`（等同于 `--sandbox workspace-write` + `--ask-for-approval on-failure`）       | Codex 可以读取文件、进行编辑并在工作区中运行命令。当沙箱命令失败或需要升级时，Codex 需要批准。                   |
| YOLO（不推荐）               | `--dangerously-bypass-approvals-and-sandbox`（别名：`--yolo`）                              | 无沙箱；无提示                                                                                                   |

> 注意：在 `workspace-write` 模式下，默认情况下禁用网络，除非在配置中启用（`[sandbox_workspace_write].network_access = true`）。

#### 在 `config.toml` 中微调

```toml
# 审批模式
approval_policy = "untrusted"
sandbox_mode    = "read-only"

# 完全自动模式
approval_policy = "on-request"
sandbox_mode    = "workspace-write"

# 可选：在 workspace-write 模式下允许网络
[sandbox_workspace_write]
network_access = true
```

您还可以将预设保存为**配置文件**：

```toml
[profiles.full_auto]
approval_policy = "on-request"
sandbox_mode    = "workspace-write"

[profiles.readonly_quiet]
approval_policy = "never"
sandbox_mode    = "read-only"
```

### 试验 Codex 沙箱

为了测试在 Codex 提供的沙箱下运行命令时会发生什么，我们在 Codex CLI 中提供了以下子命令：

```
# macOS
codex sandbox macos [--full-auto] [COMMAND]...

# Linux
codex sandbox linux [--full-auto] [COMMAND]...

# 旧版别名
codex debug seatbelt [--full-auto] [COMMAND]...
codex debug landlock [--full-auto] [COMMAND]...
```

### 平台沙箱详细信息

Codex 用于实现沙箱策略的机制取决于您的操作系统：

- **macOS 12+** 使用 **Apple Seatbelt**，并使用 `sandbox-exec` 运行命令，配置文件（`-p`）对应于指定的 `--sandbox`。
- **Linux** 使用 Landlock/seccomp API 的组合来强制执行 `sandbox` 配置。

请注意，在 Docker 等容器化环境中运行 Linux 时，如果主机/容器配置不支持必要的 Landlock/seccomp API，沙箱可能无法工作。在这种情况下，我们建议配置您的 Docker 容器，使其提供您所需的沙箱保证，然后在容器内使用 `--sandbox danger-full-access`（或更简单地说，`--dangerously-bypass-approvals-and-sandbox` 标志）运行 `codex`。
