# Tavily Web Skill

[![Codex Skill](https://img.shields.io/badge/Codex-Skill-blue)](https://github.com/openai/codex)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

简体中文 | [English](./README.md)

使用 Tavily API 完成：

- Search the web（`/search`）
- Extract webpages（`/extract`）
- Crawl webpages（`/crawl`）
- Map webpages（`/map`）
- Create Research Task（`/research`）

## 安装

将技能目录复制到你的 agent skills 目录，例如：

```bash
cp -R .claude/skills/tavily-web ~/.codex/skills/tavily-web
```

## 配置

在 `.claude/skills/tavily-web/.env` 中配置（或设置环境变量）：

```bash
TAVILY_API_KEY=your_api_key_here
```

## 快速自检

```bash
node .claude/skills/tavily-web/tavily-api.js --help
```

