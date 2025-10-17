<!-- 翻译信息
原文档: faq.md
上游提交: c32e9cfe8699110a9f317869c27f9cb34f1cf475
最后同步: 2025-10-17
翻译状态: 已完成
-->

## 常见问题（FAQ）

### OpenAI 在 2021 年发布了一个名为 Codex 的模型 - 这是否相关？

2021 年，OpenAI 发布了 Codex，这是一个旨在从自然语言提示生成代码的 AI 系统。原始的 Codex 模型已于 2023 年 3 月弃用，与此 CLI 工具是分开的。

### 支持哪些模型？

我们建议将 Codex 与 GPT-5（我们最好的编码模型）一起使用。默认推理级别为中等，您可以使用 `/model` 命令将其升级为高级别以处理复杂任务。

您也可以通过使用基于 API 的身份验证并使用 `--model` 标志启动 codex 来使用较旧的模型。

### 为什么 `o3` 或 `o4-mini` 对我不起作用？

您的 [API 帐户可能需要验证](https://help.openai.com/en/articles/10910291-api-organization-verification)才能开始流式传输响应并从 API 查看思维链摘要。如果您仍然遇到问题，请告诉我们！

### 如何阻止 Codex 编辑我的文件？

默认情况下，Codex 可以修改当前工作目录中的文件（Auto 模式）。要防止编辑，请使用 CLI 标志 `--sandbox read-only` 以只读模式运行 `codex`。或者，您可以在对话中使用 `/approvals` 更改批准级别。

### 它在 Windows 上可以运行吗？

直接在 Windows 上运行 Codex 可能可行，但不受官方支持。我们建议使用 [Windows Subsystem for Linux (WSL2)](https://learn.microsoft.com/en-us/windows/wsl/install)。
