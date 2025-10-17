<!-- 翻译信息
原文档: README.md
上游提交: 721003c552747249bad413763f436c7f4b67e41d
最后同步: 2025-10-17
翻译状态: 已完成
-->

<p align="center"><code>npm i -g @openai/codex</code><br />或 <code>brew install codex</code></p>

<p align="center"><strong>Codex CLI</strong> 是 OpenAI 推出的编码智能体，可在您的计算机上本地运行。
</br>
</br>如果您想在代码编辑器（VS Code、Cursor、Windsurf）中使用 Codex，请<a href="https://developers.openai.com/codex/ide">在 IDE 中安装</a>
</br>如果您需要 OpenAI 的<em>云端智能体</em> <strong>Codex Web</strong>，请访问 <a href="https://chatgpt.com/codex">chatgpt.com/codex</a></p>

<p align="center">
  <img src="./.github/codex-cli-splash.png" alt="Codex CLI splash" width="80%" />
  </p>

---

## 快速开始

### 安装和运行 Codex CLI

使用您喜欢的包管理器进行全局安装。如果使用 npm：

```shell
npm install -g @openai/codex
```

或者，如果使用 Homebrew：

```shell
brew install codex
```

然后只需运行 `codex` 即可开始使用：

```shell
codex
```

<details>
<summary>您也可以访问<a href="https://github.com/openai/codex/releases/latest">最新的 GitHub Release</a>，下载适合您平台的二进制文件。</summary>

每个 GitHub Release 包含多个可执行文件，但实际上您可能只需要以下其中一个：

- macOS
  - Apple Silicon/arm64: `codex-aarch64-apple-darwin.tar.gz`
  - x86_64（较旧的 Mac 硬件）: `codex-x86_64-apple-darwin.tar.gz`
- Linux
  - x86_64: `codex-x86_64-unknown-linux-musl.tar.gz`
  - arm64: `codex-aarch64-unknown-linux-musl.tar.gz`

每个压缩包包含一个带有平台信息的可执行文件（例如 `codex-x86_64-unknown-linux-musl`），因此您可能需要在解压后将其重命名为 `codex`。

</details>

### 使用 ChatGPT 套餐

<p align="center">
  <img src="./.github/codex-cli-login.png" alt="Codex CLI login" width="80%" />
  </p>

运行 `codex` 并选择 **Sign in with ChatGPT**（使用 ChatGPT 登录）。我们建议登录您的 ChatGPT 账户，以将 Codex 作为 Plus、Pro、Team、Edu 或 Enterprise 套餐的一部分使用。[详细了解您的 ChatGPT 套餐包含的内容](https://help.openai.com/en/articles/11369540-codex-in-chatgpt)。

您也可以使用 API 密钥来使用 Codex，但这需要[额外的设置](./docs/authentication.md#usage-based-billing-alternative-use-an-openai-api-key)。如果您之前使用 API 密钥进行按使用量计费，请参阅[迁移步骤](./docs/authentication.md#migrating-from-usage-based-billing-api-key)。如果您在登录时遇到问题，请在[此 issue](https://github.com/openai/codex/issues/1243) 下留言。

### 模型上下文协议（MCP）

Codex 可以访问 MCP 服务器。要配置它们，请参考[配置文档](./docs/config.md#mcp_servers)。

### 配置

Codex CLI 支持丰富的配置选项，配置文件存储在 `~/.codex/config.toml`。有关完整的配置选项，请参阅[配置文档](./docs/config.md)。

---

### 文档与常见问题

- [**快速开始**](./docs/getting-started.md)
  - [CLI 使用方法](./docs/getting-started.md#cli-usage)
  - [使用提示词作为输入运行](./docs/getting-started.md#running-with-a-prompt-as-input)
  - [示例提示词](./docs/getting-started.md#example-prompts)
  - [通过 AGENTS.md 使用记忆功能](./docs/getting-started.md#memory-with-agentsmd)
  - [配置](./docs/config.md)
- [**沙箱与权限审批**](./docs/sandbox.md)
- [**身份验证**](./docs/authentication.md)
  - [认证方法](./docs/authentication.md#forcing-a-specific-auth-method-advanced)
  - [在"无头"机器上登录](./docs/authentication.md#connecting-on-a-headless-machine)
- **自动化 Codex**
  - [GitHub Action](https://github.com/openai/codex-action)
  - [TypeScript SDK](./sdk/typescript/README.md)
  - [非交互模式（`codex exec`）](./docs/exec.md)
- [**高级主题**](./docs/advanced.md)
  - [追踪/详细日志](./docs/advanced.md#tracing--verbose-logging)
  - [模型上下文协议（MCP）](./docs/advanced.md#model-context-protocol-mcp)
- [**零数据留存（ZDR）**](./docs/zdr.md)
- [**贡献指南**](./docs/contributing.md)
- [**安装与构建**](./docs/install.md)
  - [系统要求](./docs/install.md#system-requirements)
  - [DotSlash](./docs/install.md#dotslash)
  - [从源码构建](./docs/install.md#build-from-source)
- [**常见问题（FAQ）**](./docs/faq.md)
- [**开源基金**](./docs/open-source-fund.md)

---

## 许可证

本仓库采用 [Apache-2.0 许可证](LICENSE)。
