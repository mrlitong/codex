<!-- 翻译信息
原文档: sdk/typescript/README.md
上游提交: 1772bf63
最后同步: 2025-10-17
翻译状态: 已完成
-->

# Codex SDK

将 Codex 代理嵌入你的工作流和应用程序中。

TypeScript SDK 封装了打包的 `codex` 二进制文件。它生成 CLI 进程并通过 stdin/stdout 交换 JSONL 事件。

## 安装

```bash
npm install @openai/codex-sdk
```

需要 Node.js 18+。

## 快速开始

```typescript
import { Codex } from "@openai/codex-sdk";

const codex = new Codex();
const thread = codex.startThread();
const turn = await thread.run("Diagnose the test failure and propose a fix");

console.log(turn.finalResponse);
console.log(turn.items);
```

在同一个 `Thread` 实例上重复调用 `run()` 以继续对话。

```typescript
const nextTurn = await thread.run("Implement the fix");
```

### 流式响应

`run()` 会缓冲事件直到回合结束。要对中间进度做出反应——工具调用、流式响应和文件差异——请改用 `runStreamed()`，它返回结构化事件的异步生成器。

```typescript
const { events } = await thread.runStreamed("Diagnose the test failure and propose a fix");

for await (const event of events) {
  switch (event.type) {
    case "item.completed":
      console.log("item", event.item);
      break;
    case "turn.completed":
      console.log("usage", event.usage);
      break;
  }
}
```

### 结构化输出

Codex 代理可以生成符合指定架构的 JSON 响应。可以为每个回合提供纯 JSON 对象作为架构。

```typescript
const schema = {
  type: "object",
  properties: {
    summary: { type: "string" },
    status: { type: "string", enum: ["ok", "action_required"] },
  },
  required: ["summary", "status"],
  additionalProperties: false,
} as const;

const turn = await thread.run("Summarize repository status", { outputSchema: schema });
console.log(turn.finalResponse);
```

你还可以使用 [`zod-to-json-schema`](https://www.npmjs.com/package/zod-to-json-schema) 包从 [Zod 架构](https://github.com/colinhacks/zod) 创建 JSON 架构，并将 `target` 设置为 `"openAi"`。

```typescript
const schema = z.object({
  summary: z.string(),
  status: z.enum(["ok", "action_required"]),
});

const turn = await thread.run("Summarize repository status", {
  outputSchema: zodToJsonSchema(schema, { target: "openAi" }),
});
console.log(turn.finalResponse);
```

### 恢复现有线程

线程保存在 `~/.codex/sessions` 中。如果你丢失了内存中的 `Thread` 对象，可以使用 `resumeThread()` 重建它并继续。

```typescript
const savedThreadId = process.env.CODEX_THREAD_ID!;
const thread = codex.resumeThread(savedThreadId);
await thread.run("Implement the fix");
```

### 工作目录控制

Codex 默认在当前工作目录中运行。为避免不可恢复的错误，Codex 要求工作目录是一个 Git 仓库。你可以在创建线程时通过传递 `skipGitRepoCheck` 选项来跳过 Git 仓库检查。

```typescript
const thread = codex.startThread({
  workingDirectory: "/path/to/project",
  skipGitRepoCheck: true,
});
```
