# Tavily Web Skill

[![Codex Skill](https://img.shields.io/badge/Codex-Skill-blue)](https://github.com/openai/codex)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

English | [简体中文](./README_CN.md)

Use Tavily API to:

- Search the web (`/search`)
- Extract webpages (`/extract`)
- Crawl webpages (`/crawl`)
- Map webpages (`/map`)
- Create research tasks (`/research`)

## Installation

Copy the skill folder to your agent's skills directory, e.g.:

```bash
cp -R .claude/skills/tavily-web ~/.codex/skills/tavily-web
```

## Configuration

Create `.env` at `.claude/skills/tavily-web/.env` (or set env var):

```bash
TAVILY_API_KEY=your_api_key_here
```

## Quick Test

```bash
node .claude/skills/tavily-web/tavily-api.js --help
```

