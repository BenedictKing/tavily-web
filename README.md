# Tavily Web Skill

[![Claude Code Skill](https://img.shields.io/badge/Claude%20Code-Skill-blue)](https://skillsmp.com/)
[![GitHub Stars](https://img.shields.io/github/stars/BenedictKing/tavily-web?style=social)](https://github.com/BenedictKing/tavily-web)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

English | [简体中文](./README_CN.md)

> 🌐 Powerful web search, extraction, crawling, and research capabilities for Claude Code using Tavily API!

## Introduction

Tavily Web Skill is a comprehensive Claude Code skill that provides advanced web interaction capabilities through the Tavily API. It enables intelligent web search, content extraction, site crawling, URL discovery, and structured research tasks.

### Key Features

- 🔍 **Web Search**: Perform intelligent web searches with customizable parameters
- 📄 **Content Extraction**: Extract clean content from specific URLs
- 🕷️ **Web Crawling**: Crawl websites to discover and extract content
- 🗺️ **Site Mapping**: Discover all URLs within a website
- 📊 **Research Tasks**: Create structured research reports on any topic
- 🎯 **Smart Triggers**: Automatically activates when web research is needed
- 🌍 **Bilingual Support**: Supports both English and Chinese trigger keywords

## Quick Start

Set up in 5 minutes

## Installation

### Option 1: Install via skills CLI (Recommended)

The easiest way to install this skill is using the `skills` CLI tool:

```bash
# Install globally to all detected agents (Claude Code, Cursor, Codex, etc.)
npx skills add -g BenedictKing/tavily-web

# Or install to current project only
npx skills add BenedictKing/tavily-web
```

The skill will be automatically installed to `~/.claude/skills/tavily-web` and loaded by Claude Code.

### Option 2: Manual Installation via Git Clone

If you prefer manual installation or want to customize the setup:

#### 1. Clone the Repository

```bash
# Clone to Claude Code's skills directory
git clone https://github.com/BenedictKing/tavily-web.git ~/.claude/skills/tavily-web

# Or clone to your preferred location
git clone https://github.com/BenedictKing/tavily-web.git
cd tavily-web
```

#### 2. Get API Key

Visit [tavily.com](https://tavily.com) to register and get your API key.

#### 3. Configure API Key

Create a `.env` file in the skill directory:

```bash
cd .claude/skills/tavily-web
cp .env.example .env
```

Edit the `.env` file and add your API key:

```bash
TAVILY_API_KEY=your_actual_api_key_here
```

#### 4. Test the Script

Verify your configuration:

```bash
# Search the web
node .claude/skills/tavily-web/tavily-api.js search "latest AI developments"

# Extract content from a URL
node .claude/skills/tavily-web/tavily-api.js extract "https://example.com"
```

If you see JSON responses, your setup is successful!

## Usage

The skill can be invoked manually or activates automatically when web research is needed.

### Manual Invocation

Use the skill name directly:

```
You: /tavily-web search "React 19 new features"
```

### Auto-Trigger

The skill automatically activates when detecting these keywords:

**Search Queries**
- Chinese: 搜索、查找、网页搜索
- English: search, find, web search, look up

**Content Extraction**
- Chinese: 提取、抓取内容
- English: extract, fetch content, get content

**Web Crawling**
- Chinese: 爬取、遍历网站
- English: crawl, spider, scrape

**Research Tasks**
- Chinese: 研究、调研、分析
- English: research, investigate, analyze

## Available Commands

### 1. Search (`search`)

Perform intelligent web searches:

```bash
node tavily-api.js search "query" [options]

Options:
  --max-results <n>     Maximum number of results (default: 5)
  --include-domains     Comma-separated domains to include
  --exclude-domains     Comma-separated domains to exclude
  --search-depth        Search depth: basic or advanced
```

**Example:**
```
You: Search for "Next.js 15 middleware examples"
Claude: [Automatically calls Tavily search API]
```

### 2. Extract (`extract`)

Extract clean content from URLs:

```bash
node tavily-api.js extract "url1,url2,url3"
```

**Example:**
```
You: Extract content from https://example.com/article
Claude: [Automatically calls Tavily extract API]
```

### 3. Crawl (`crawl`)

Crawl websites to discover and extract content:

```bash
node tavily-api.js crawl "url" [options]

Options:
  --max-pages <n>       Maximum pages to crawl (default: 10)
  --include-patterns    URL patterns to include
  --exclude-patterns    URL patterns to exclude
```

**Example:**
```
You: Crawl https://docs.example.com for documentation
Claude: [Automatically calls Tavily crawl API]
```

### 4. Map (`map`)

Discover all URLs within a website:

```bash
node tavily-api.js map "url" [options]

Options:
  --max-urls <n>        Maximum URLs to discover (default: 100)
  --filter-pattern      Pattern to filter URLs
```

**Example:**
```
You: Map all URLs on https://example.com
Claude: [Automatically calls Tavily map API]
```

### 5. Research (`research`)

Create structured research reports:

```bash
node tavily-api.js research "topic" [options]

Options:
  --max-sources <n>     Maximum sources to use (default: 10)
  --report-format       Format: summary, detailed, or academic
```

**Example:**
```
You: Research "AI safety best practices"
Claude: [Automatically calls Tavily research API]
```

## How It Works

### Auto-Trigger Mechanism

The skill automatically activates when detecting:

**Web Research Needs**
- Questions requiring current information
- Requests to search or find information online
- Content extraction from specific URLs
- Website crawling or mapping tasks

**Trigger Keywords**
- Chinese: 搜索、查资料、最新、网页、爬取、研究
- English: search, find, latest, web, crawl, research, extract

## FAQ

### Q: Is an API key required?
A: Yes, you need a Tavily API key. Visit [tavily.com](https://tavily.com) to get one.

### Q: Where should I put the .env file?
A: Place it at `.claude/skills/tavily-web/.env`

### Q: How do I know if the skill is working?
A: When you ask questions requiring web research, Claude will automatically call the Tavily API. You'll see up-to-date information in the responses.

### Q: What's the difference between search and research?
A:
- **Search**: Returns raw search results with URLs and snippets
- **Research**: Creates a structured report with synthesized information from multiple sources

### Q: Can I limit searches to specific domains?
A: Yes, use the `--include-domains` option to specify allowed domains.

## Example Conversations

### Example 1: Web Search
```
You: What are the latest developments in AI in 2026?

Claude: [Automatically calls Tavily search API]
Based on recent web sources...
[Provides current information with citations]
```

### Example 2: Content Extraction
```
You: Extract the main content from https://example.com/article

Claude: [Automatically calls Tavily extract API]
Here's the extracted content...
[Provides clean, formatted content]
```

### Example 3: Research Task
```
You: Research the best practices for React performance optimization

Claude: [Automatically calls Tavily research API]
Here's a comprehensive research report...
[Provides structured report with multiple sources]
```

## Next Steps

- Check [.claude/skills/tavily-web/SKILL.md](./.claude/skills/tavily-web/SKILL.md) for technical details
- Start asking questions and let Claude automatically fetch the latest web information!

## Troubleshooting

### Script Execution Fails
```bash
# Ensure script has execute permissions
chmod +x .claude/skills/tavily-web/tavily-api.js

# Ensure Node.js is installed
node --version  # Should display version number
```

### API Returns 401 Error
Check if API key is correctly configured:
```bash
# View .env file
cat .claude/skills/tavily-web/.env

# Ensure correct format
TAVILY_API_KEY=your_key_here  # ✅ Correct
TAVILY_API_KEY = your_key_here  # ❌ Wrong (has spaces)
```

### API Returns 429 Error
Rate limit reached. Wait for some time or upgrade your API key quota.

---

🎉 Setup complete! Now you can enjoy powerful web research capabilities!
