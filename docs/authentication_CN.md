<!-- 翻译信息
原文档: authentication.md
上游提交: c32e9cfe8699110a9f317869c27f9cb34f1cf475
最后同步: 2025-10-17
翻译状态: 已完成
-->

# 身份验证

## 按使用量计费的替代方案：使用 OpenAI API 密钥

如果您更喜欢按使用量付费，您仍然可以使用 OpenAI API 密钥进行身份验证：

```shell
printenv OPENAI_API_KEY | codex login --with-api-key
```

或者，从文件读取：

```shell
codex login --with-api-key < my_key.txt
```

旧的 `--api-key` 标志现在会退出并显示错误，指示您使用 `--with-api-key`，这样密钥就不会出现在 shell 历史记录或进程列表中。

此密钥至少必须具有 Responses API 的写入权限。

## 从 API 密钥迁移到 ChatGPT 登录

如果您之前通过 API 密钥使用按使用量计费的方式使用 Codex CLI，并希望切换到使用 ChatGPT 套餐，请按照以下步骤操作：

1. 更新 CLI 并确保 `codex --version` 为 `0.20.0` 或更高版本
2. 删除 `~/.codex/auth.json`（Windows 上：`C:\\Users\\USERNAME\\.codex\\auth.json`）
3. 再次运行 `codex login`

## 在"无头"机器上连接

目前，登录过程需要在 `localhost:1455` 上运行服务器。如果您在"无头"服务器上（例如 Docker 容器或通过 `ssh` 连接到远程机器），在本地机器的浏览器中加载 `localhost:1455` 不会自动连接到在_无头_机器上运行的 Web 服务器，因此您必须使用以下解决方法之一：

### 在本地进行身份验证并将凭据复制到"无头"机器

最简单的解决方案可能是在本地机器上完成 `codex login` 过程，这样 `localhost:1455` _可以_在您的 Web 浏览器中访问。当您完成身份验证过程后，`auth.json` 文件应该在 `$CODEX_HOME/auth.json` 位置可用（在 Mac/Linux 上，`$CODEX_HOME` 默认为 `~/.codex`，而在 Windows 上，默认为 `%USERPROFILE%\\.codex`）。

因为 `auth.json` 文件不绑定到特定主机，所以一旦您在本地完成身份验证流程，就可以将 `$CODEX_HOME/auth.json` 文件复制到无头机器上，然后 `codex` 应该在该机器上"正常工作"。注意要将文件复制到 Docker 容器，您可以执行：

```shell
# 将 MY_CONTAINER 替换为您的 Docker 容器的名称或 ID：
CONTAINER_HOME=$(docker exec MY_CONTAINER printenv HOME)
docker exec MY_CONTAINER mkdir -p "$CONTAINER_HOME/.codex"
docker cp auth.json MY_CONTAINER:"$CONTAINER_HOME/.codex/auth.json"
```

而如果您通过 `ssh` 连接到远程机器，您可能想使用 [`scp`](https://en.wikipedia.org/wiki/Secure_copy_protocol)：

```shell
ssh user@remote 'mkdir -p ~/.codex'
scp ~/.codex/auth.json user@remote:~/.codex/auth.json
```

或者尝试这个一行命令：

```shell
ssh user@remote 'mkdir -p ~/.codex && cat > ~/.codex/auth.json' < ~/.codex/auth.json
```

### 通过 VPS 或远程连接

如果您在没有本地浏览器的远程机器（VPS/服务器）上运行 Codex，登录助手会在远程主机的 `localhost:1455` 上启动服务器。要在本地浏览器中完成登录，在开始登录流程之前将该端口转发到您的机器：

```bash
# 从您的本地机器
ssh -L 1455:localhost:1455 <user>@<remote-host>
```

然后，在该 SSH 会话中，运行 `codex` 并选择"Sign in with ChatGPT"（使用 ChatGPT 登录）。出现提示时，在本地浏览器中打开打印的 URL（将是 `http://localhost:1455/...`）。流量将通过隧道传输到远程服务器。
