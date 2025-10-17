<!-- 翻译信息
原文档: PNPM.md
上游提交: 1772bf63
最后同步: 2025-10-17
翻译状态: 已完成
-->

# 迁移到 pnpm

本项目已从 npm 迁移到 pnpm，以改善依赖管理和开发体验。

## 为什么使用 pnpm？

- **更快的安装速度**: pnpm 比 npm 和 yarn 快得多
- **节省磁盘空间**: pnpm 使用内容寻址存储来避免重复
- **防止幽灵依赖**: pnpm 创建严格的 node_modules 结构
- **原生工作空间支持**: 简化 monorepo 管理

## 如何使用 pnpm

### 安装

```bash
# 全局安装 pnpm
npm install -g pnpm@10.8.1

# 或使用 corepack（Node.js 22+ 可用）
corepack enable
corepack prepare pnpm@10.8.1 --activate
```

### 常用命令

| npm 命令        | pnpm 等效命令    |
| --------------- | ---------------- |
| `npm install`   | `pnpm install`   |
| `npm run build` | `pnpm run build` |
| `npm test`      | `pnpm test`      |
| `npm run lint`  | `pnpm run lint`  |

### 工作空间专用命令

| 操作                       | 命令                                     |
| -------------------------- | ---------------------------------------- |
| 在特定包中运行命令         | `pnpm --filter @openai/codex run build`  |
| 在特定包中安装依赖         | `pnpm --filter @openai/codex add lodash` |
| 在所有包中运行命令         | `pnpm -r run test`                       |

## Monorepo 结构

```
codex/
├── pnpm-workspace.yaml    # 工作空间配置
├── .npmrc                 # pnpm 配置
├── package.json           # 根依赖和脚本
├── codex-cli/             # 主包
│   └── package.json       # codex-cli 特定依赖
└── docs/                  # 文档（未来的包）
```

## 配置文件

- **pnpm-workspace.yaml**: 定义 monorepo 中包含的包
- **.npmrc**: 配置 pnpm 行为
- **根 package.json**: 包含共享脚本和依赖

## CI/CD

CI/CD 工作流已更新为使用 pnpm 而不是 npm。确保你的 CI 环境使用 pnpm 10.8.1 或更高版本。

## 已知问题

如果你在使用 pnpm 时遇到问题，请尝试以下解决方案：

1. 删除 `node_modules` 文件夹和 `pnpm-lock.yaml` 文件，然后运行 `pnpm install`
2. 确保你使用的是 pnpm 10.8.1 或更高版本
3. 验证已安装 Node.js 22 或更高版本
