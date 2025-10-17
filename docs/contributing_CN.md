<!-- 翻译信息
原文档: docs/contributing.md
上游提交: 1772bf63
最后同步: 2025-10-17
翻译状态: 已完成
-->

## 贡献指南

本项目正在积极开发中，代码可能会发生重大变化。

**目前，我们仅计划优先审查外部贡献中的 bug 修复或安全修复。**

如果你想添加新功能或更改现有功能的行为，请先开启 issue 提出该功能，并在投入时间构建之前获得 OpenAI 团队成员的批准。

**未经此流程的新贡献可能会被关闭**，如果它们与我们当前的路线图不一致或与其他优先事项/即将推出的功能冲突。

### 开发工作流

- 从 `main` 创建一个 _主题分支_ - 例如 `feat/interactive-prompt`。
- 保持你的更改聚焦。多个不相关的修复应作为单独的 PR 提交。
- 按照上面的[开发设置](#development-workflow)说明，确保你的更改没有 lint 警告和测试失败。

### 编写高影响力的代码更改

1. **从 issue 开始。** 开启新 issue 或在现有讨论中评论，以便我们在编写代码之前就解决方案达成一致。
2. **添加或更新测试。** 每个新功能或 bug 修复都应该配备测试覆盖，在你的更改之前失败，之后通过。不要求 100% 覆盖率，但要力求有意义的断言。
3. **记录行为。** 如果你的更改影响面向用户的行为，请更新 README、内联帮助（`codex --help`）或相关示例项目。
4. **保持提交原子化。** 每个提交都应该可以编译，并且测试应该通过。这使审查和潜在的回滚更容易。

### 开启 Pull Request

- 填写 PR 模板（或包含类似信息）- **是什么？为什么？如何做？**
- 在本地运行**所有**检查（`cargo test && cargo clippy --tests && cargo fmt -- --config imports_granularity=Item`）。可以在本地捕获的 CI 失败会拖慢流程。
- 确保你的分支与 `main` 保持最新，并且已解决合并冲突。
- 仅当你认为 PR 处于可合并状态时，才将其标记为 **Ready for review**。

### 审查流程

1. 一位维护者将被指派为主要审查者。
2. 如果你的 PR 添加了之前未讨论和批准的新功能，我们可能会选择关闭你的 PR（参见[贡献指南](#contributing)）。
3. 我们可能会要求更改 - 请不要把这当作针对你个人。我们重视这项工作，但我们也重视一致性和长期可维护性。
4. 当达成共识认为 PR 符合标准时，维护者将进行 squash-and-merge。

### 社区价值观

- **友善和包容。** 尊重他人；我们遵循[贡献者公约](https://www.contributor-covenant.org/)。
- **假定善意。** 书面交流很难 - 要宽容大度。
- **教学相长。** 如果你发现某些令人困惑的内容，请开启 issue 或 PR 进行改进。

### 获取帮助

如果你在设置项目时遇到问题，想要获得对某个想法的反馈，或者只是想打个招呼 - 请开启 Discussion 或跳转到相关 issue。我们很乐意提供帮助。

让我们一起将 Codex CLI 打造成一个令人难以置信的工具。**Happy hacking!** :rocket:

### 贡献者许可协议 (CLA)

所有贡献者**必须**接受 CLA。流程很轻松：

1. 打开你的 Pull Request。
2. 粘贴以下评论（如果之前已签署，回复 `recheck`）：

   ```text
   I have read the CLA Document and I hereby sign the CLA
   ```

3. CLA-Assistant 机器人会在仓库中记录你的签名，并将状态检查标记为通过。

无需特殊的 Git 命令、电子邮件附件或提交页脚。

#### 快速修复

| 场景               | 命令                                             |
| ------------------ | ------------------------------------------------ |
| 修改最后一次提交   | `git commit --amend -s --no-edit && git push -f` |

**DCO 检查**会阻止合并，直到 PR 中的每个提交都带有页脚（使用 squash 时只需一个）。

### 发布 `codex`

_仅限管理员。_

确保你在 `main` 分支上并且没有本地更改。然后运行：

```shell
VERSION=0.2.0  # 也可以是 0.2.0-alpha.1 或任何有效的 Rust 版本。
./codex-rs/scripts/create_github_release.sh "$VERSION"
```

这将在 `main` 之上创建一个本地提交，在 `codex-rs/Cargo.toml` 中将 `version` 设置为 `$VERSION`（注意在 `main` 上，我们将版本保留为 `version = "0.0.0"`）。

这将使用标签 `rust-v${VERSION}` 推送提交，进而触发[发布工作流](../.github/workflows/rust-release.yml)。这将创建一个名为 `$VERSION` 的新 GitHub Release。

如果生成的 GitHub Release 一切正常，取消选中 **pre-release** 框，使其成为最新版本。

创建 PR 以更新 Homebrew 上的 [`Formula/c/codex.rb`](https://github.com/Homebrew/homebrew-core/blob/main/Formula/c/codex.rb)。

### 安全与负责任的 AI

你发现了漏洞或对模型输出有疑虑？请发送电子邮件至 **security@openai.com**，我们将及时回复。
