---
name: tavily-fetcher
version: 1.0.0
author: petaflops
description: 执行 Tavily API 调用的独立子任务（内部使用）
allowed-tools:
  - Bash
context: fork
---

# Tavily Fetcher 子技能

> 注意：这是一个内部子技能，由 `tavily-web` 主技能通过 Task 工具调用。

## 用途

在 `context: fork` 的独立上下文中执行 Tavily API 调用，避免携带主对话上下文，降低 token 消耗。

## 接收参数

通过 Task 的 `prompt` 传入完整命令，约定使用 stdin 传 JSON：

```bash
cat <<'JSON' | node .claude/skills/tavily-web/tavily-api.js <search|extract|crawl|map|research>
{ ...payload... }
JSON
```

## 输出

原样返回 Tavily API 的 JSON 响应（pretty print）。

