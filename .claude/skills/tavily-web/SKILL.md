---
name: tavily-web
version: 1.0.0
author: petaflops
description: 使用 Tavily API 进行网页搜索（search）、页面抽取（extract）、站点遍历（crawl）、站点 URL 发现（map）与结构化研究任务（research）。适用于需要最新信息、需要从指定 URL 提取内容、或需要对站点做自动遍历/页面发现的场景。触发词：tavily、搜索网页、查资料、最新、web search、extract、crawl、map、research
allowed-tools:
  - Task
  - Bash
  - Read
  - Write
user-invocable: true
---

# Tavily Web Skill

## 触发与选型

根据用户意图选择 Tavily 端点：

- **search**：需要“搜索网页/最新信息/找来源/找链接”
- **extract**：已给出 URL，需要抽取/总结正文
- **crawl**：需要按指令遍历站点并抓取页面内容
- **map**：需要发现站点页面列表/站点结构（不抓全文或只抓元信息）
- **research**：需要按给定 `output_schema` 产出结构化研究结果

## 推荐架构（主技能 + 子技能）

遵循 `context7-auto-research` 的成熟模式：

1. **主技能（当前上下文）**：理解用户问题 → 选择端点 → 组装 JSON payload
2. **子技能（fork 上下文）**：只负责执行 HTTP 调用，避免携带对话历史浪费 token

## 执行方式

使用 Task 工具调用 `tavily-fetcher` 子技能，传入命令与 JSON（stdin）：

```
Task 参数：
- subagent_type: Bash
- description: "Call Tavily API"
- prompt: cat <<'JSON' | node .claude/skills/tavily-web/tavily-api.js <search|extract|crawl|map|research>
  { ...payload... }
  JSON
```

## Payload 示例（对应你提供的 curl）

### 1) Search the web

```bash
cat <<'JSON' | node .claude/skills/tavily-web/tavily-api.js search
{
  "query": "who is Leo Messi?",
  "auto_parameters": false,
  "topic": "general",
  "search_depth": "basic",
  "chunks_per_source": 3,
  "max_results": 1,
  "time_range": null,
  "start_date": "2025-02-09",
  "end_date": "2025-12-29",
  "include_answer": false,
  "include_raw_content": false,
  "include_images": false,
  "include_image_descriptions": false,
  "include_favicon": false,
  "include_domains": [],
  "exclude_domains": [],
  "country": null,
  "include_usage": false
}
JSON
```

### 2) Extract webpages

```bash
cat <<'JSON' | node .claude/skills/tavily-web/tavily-api.js extract
{
  "urls": "https://en.wikipedia.org/wiki/Artificial_intelligence",
  "query": "<string>",
  "chunks_per_source": 3,
  "extract_depth": "basic",
  "include_images": false,
  "include_favicon": false,
  "format": "markdown",
  "timeout": "None",
  "include_usage": false
}
JSON
```

### 3) Crawl webpages

```bash
cat <<'JSON' | node .claude/skills/tavily-web/tavily-api.js crawl
{
  "url": "docs.tavily.com",
  "instructions": "Find all pages about the Python SDK",
  "chunks_per_source": 3,
  "max_depth": 1,
  "max_breadth": 20,
  "limit": 50,
  "select_paths": null,
  "select_domains": null,
  "exclude_paths": null,
  "exclude_domains": null,
  "allow_external": true,
  "include_images": false,
  "extract_depth": "basic",
  "format": "markdown",
  "include_favicon": false,
  "timeout": 150,
  "include_usage": false
}
JSON
```

### 4) Map webpages

```bash
cat <<'JSON' | node .claude/skills/tavily-web/tavily-api.js map
{
  "url": "docs.tavily.com",
  "instructions": "Find all pages about the Python SDK",
  "max_depth": 1,
  "max_breadth": 20,
  "limit": 50,
  "select_paths": null,
  "select_domains": null,
  "exclude_paths": null,
  "exclude_domains": null,
  "allow_external": true,
  "timeout": 150,
  "include_usage": false
}
JSON
```

### 5) Create Research Task

```bash
cat <<'JSON' | node .claude/skills/tavily-web/tavily-api.js research
{
  "input": "What are the latest developments in AI?",
  "model": "auto",
  "stream": false,
  "output_schema": {
    "properties": {
      "company": {
        "type": "string",
        "description": "The name of the company"
      },
      "key_metrics": {
        "type": "array",
        "description": "List of key performance metrics",
        "items": {
          "type": "string"
        }
      },
      "financial_details": {
        "type": "object",
        "description": "Detailed financial breakdown",
        "properties": {
          "operating_income": {
            "type": "number",
            "description": "Operating income for the period"
          }
        }
      }
    },
    "required": [
      "company"
    ]
  },
  "citation_format": "numbered"
}
JSON
```

## 环境变量与密钥

支持两种方式配置 API Key（优先级：环境变量 > `.env`）：

1. 环境变量：`TAVILY_API_KEY`
2. `.env` 文件：放在 `.claude/skills/tavily-web/.env`，可从 `.env.example` 复制

