<!-- 翻译信息
原文档: install.md
上游提交: c32e9cfe8699110a9f317869c27f9cb34f1cf475
最后同步: 2025-10-17
翻译状态: 已完成
-->

## 安装与构建

### 系统要求

| 要求                  | 详细信息                                                    |
| --------------------- | ----------------------------------------------------------- |
| 操作系统              | macOS 12+、Ubuntu 20.04+/Debian 10+ 或 Windows 11 **（通过 WSL2）** |
| Git（可选，推荐）     | 2.23+ 以支持内置的 PR 辅助功能                             |
| RAM                   | 最低 4 GB（推荐 8 GB）                                      |

### DotSlash

GitHub Release 中还包含一个名为 `codex` 的 [DotSlash](https://dotslash-cli.com/) 文件用于 Codex CLI。使用 DotSlash 文件可以轻松地将可执行文件提交到源代码控制中，以确保所有贡献者使用相同版本的可执行文件，无论他们使用什么平台进行开发。

### 从源码构建

```bash
# 克隆仓库并导航到 Cargo 工作区的根目录
git clone https://github.com/openai/codex.git
cd codex/codex-rs

# 如有必要，安装 Rust 工具链
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
source "$HOME/.cargo/env"
rustup component add rustfmt
rustup component add clippy

# 构建 Codex
cargo build

# 使用示例提示词启动 TUI
cargo run --bin codex -- "explain this codebase to me"

# 进行更改后，确保代码整洁
cargo fmt -- --config imports_granularity=Item
cargo clippy --tests

# 运行测试
cargo test
```
