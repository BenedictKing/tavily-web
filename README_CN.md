# Tavily Web Skill

[![Claude Code Skill](https://img.shields.io/badge/Claude%20Code-Skill-blue)](https://skillsmp.com/)
[![GitHub Stars](https://img.shields.io/github/stars/BenedictKing/tavily-web?style=social)](https://github.com/BenedictKing/tavily-web)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

[English](./README.md) | 简体中文

> 🌐 使用 Tavily API 为 Claude Code 提供强大的网页搜索、提取、爬取和研究能力！

## 简介

Tavily Web Skill 是一个全面的 Claude Code 技能，通过 Tavily API 提供高级网页交互能力。它支持智能网页搜索、内容提取、站点爬取、URL 发现和结构化研究任务。

### 核心特性

- 🔍 **网页搜索**：执行智能网页搜索，支持自定义参数
- 📄 **内容提取**：从指定 URL 提取干净的内容
- 🕷️ **网页爬取**：爬取网站以发现和提取内容
- 🗺️ **站点地图**：发现网站内的所有 URL
- 📊 **研究任务**：创建关于任何主题的结构化研究报告
- 🎯 **智能触发**：需要网页研究时自动激活
- 🌍 **双语支持**：支持中英文触发关键词

## 快速开始

5 分钟完成配置

## 安装方式

### 方式一：使用 add-skill 安装（推荐）

最简单的安装方式是使用 `add-skill` 工具：

```bash
# 安装到 Claude Code
npx add-skill BenedictKing/tavily-web

# 或全局安装到所有检测到的 AI 编程助手（Claude Code、Cursor、Codex 等）
npx add-skill BenedictKing/tavily-web -g
```

Skill 会自动安装到 `~/.claude/skills/tavily-web` 并被 Claude Code 加载。

### 方式二：通过 Git Clone 手动安装

如果你偏好手动安装或想自定义设置：

#### 1. 克隆仓库

```bash
# 克隆到 Claude Code 的 skills 目录
git clone https://github.com/BenedictKing/tavily-web.git ~/.claude/skills/tavily-web

# 或克隆到你偏好的位置
git clone https://github.com/BenedictKing/tavily-web.git
cd tavily-web
```

#### 2. 获取 API Key

访问 [tavily.com](https://tavily.com) 注册并获取你的 API key。

#### 3. 配置 API Key

在 skill 目录下创建 `.env` 文件：

```bash
cd .claude/skills/tavily-web
cp .env.example .env
```

编辑 `.env` 文件，填入你的 API key：

```bash
TAVILY_API_KEY=your_actual_api_key_here
```

#### 4. 测试脚本

验证配置是否正确：

```bash
# 搜索网页
node .claude/skills/tavily-web/tavily-api.cjs search "最新 AI 发展"

# 从 URL 提取内容
node .claude/skills/tavily-web/tavily-api.cjs extract "https://example.com"
```

如果看到 JSON 响应，说明配置成功！

## 使用方法

该技能可以手动调用，也可以在需要网页研究时自动激活。

### 手动调用

直接使用技能名称：

```
你：/tavily-web search "React 19 新特性"
```

### 自动触发

技能会在检测到以下关键词时自动激活：

**搜索查询**
- 中文：搜索、查找、网页搜索
- 英文：search、find、web search、look up

**内容提取**
- 中文：提取、抓取内容
- 英文：extract、fetch content、get content

**网页爬取**
- 中文：爬取、遍历网站
- 英文：crawl、spider、scrape

**研究任务**
- 中文：研究、调研、分析
- 英文：research、investigate、analyze

## 可用命令

### 1. 搜索 (`search`)

执行智能网页搜索：

```bash
node tavily-api.cjs search "查询内容" [选项]

选项：
  --max-results <n>     最大结果数量（默认：5）
  --include-domains     包含的域名（逗号分隔）
  --exclude-domains     排除的域名（逗号分隔）
  --search-depth        搜索深度：basic 或 advanced
```

**示例：**
```
你：搜索 "Next.js 15 中间件示例"
Claude：[自动调用 Tavily 搜索 API]
```

### 2. 提取 (`extract`)

从 URL 提取干净的内容：

```bash
node tavily-api.cjs extract "url1,url2,url3"
```

**示例：**
```
你：提取 https://example.com/article 的内容
Claude：[自动调用 Tavily 提取 API]
```

### 3. 爬取 (`crawl`)

爬取网站以发现和提取内容：

```bash
node tavily-api.cjs crawl "url" [选项]

选项：
  --max-pages <n>       最大爬取页面数（默认：10）
  --include-patterns    包含的 URL 模式
  --exclude-patterns    排除的 URL 模式
```

**示例：**
```
你：爬取 https://docs.example.com 的文档
Claude：[自动调用 Tavily 爬取 API]
```

### 4. 地图 (`map`)

发现网站内的所有 URL：

```bash
node tavily-api.cjs map "url" [选项]

选项：
  --max-urls <n>        最大发现 URL 数（默认：100）
  --filter-pattern      过滤 URL 的模式
```

**示例：**
```
你：映射 https://example.com 的所有 URL
Claude：[自动调用 Tavily 地图 API]
```

### 5. 研究 (`research`)

创建结构化研究报告：

```bash
node tavily-api.cjs research "主题" [选项]

选项：
  --max-sources <n>     最大使用来源数（默认：10）
  --report-format       格式：summary、detailed 或 academic
```

**示例：**
```
你：研究 "AI 安全最佳实践"
Claude：[自动调用 Tavily 研究 API]
```

## 工作原理

### 自动触发机制

技能会在检测到以下情况时自动激活：

**网页研究需求**
- 需要当前信息的问题
- 在线搜索或查找信息的请求
- 从特定 URL 提取内容
- 网站爬取或映射任务

**触发关键词**
- 中文：搜索、查资料、最新、网页、爬取、研究
- 英文：search、find、latest、web、crawl、research、extract

## 常见问题

### Q: 是否需要 API key？
A: 是的，你需要一个 Tavily API key。访问 [tavily.com](https://tavily.com) 获取。

### Q: .env 文件放在哪里？
A: 放在 `.claude/skills/tavily-web/.env`

### Q: 如何知道 skill 是否在工作？
A: 当你询问需要网页研究的问题时，Claude 会自动调用 Tavily API。你可以在响应中看到最新的信息。

### Q: search 和 research 有什么区别？
A:
- **Search（搜索）**：返回原始搜索结果，包含 URL 和摘要
- **Research（研究）**：创建结构化报告，综合多个来源的信息

### Q: 可以限制搜索特定域名吗？
A: 可以，使用 `--include-domains` 选项指定允许的域名。

## 示例对话

### 示例 1：网页搜索
```
你：2026 年 AI 领域有哪些最新发展？

Claude：[自动调用 Tavily 搜索 API]
根据最近的网页来源...
[提供带引用的当前信息]
```

### 示例 2：内容提取
```
你：提取 https://example.com/article 的主要内容

Claude：[自动调用 Tavily 提取 API]
这是提取的内容...
[提供干净、格式化的内容]
```

### 示例 3：研究任务
```
你：研究 React 性能优化的最佳实践

Claude：[自动调用 Tavily 研究 API]
这是一份全面的研究报告...
[提供包含多个来源的结构化报告]
```

## 与 Exa 的比较

| 功能 | [Tavily](https://tavily.com) | [Exa](https://exa.ai) |
|------|--------|-----|
| 搜索类型 | 关键词 + AI 增强 | 基于嵌入的语义搜索 |
| 爬取/映射 | ✅ | ❌ |
| 研究报告 | ✅ 带引用的结构化报告 | ✅ 自定义输出模式 |
| 搜索模式 | basic/advanced | neural/fast/deep/auto |
| 相似查找 | ❌ | ✅ |
| 类别搜索 | ❌ | ✅（公司、人物、论文等） |
| 成本追踪 | ❌ | ✅ 每次请求详细 |

**Claude Code Skills：**
- Tavily：[BenedictKing/tavily-web](https://github.com/BenedictKing/tavily-web)
- Exa：[BenedictKing/exa-search](https://github.com/BenedictKing/exa-search)

**使用 Tavily 当：**
- 需要网站爬取/映射
- 需要带引用的结构化研究
- 偏好关键词搜索

**使用 Exa 当：**
- 需要语义/基于嵌入的搜索
- 查找相似内容很重要
- 需要特定类别搜索
- 需要成本追踪

## 下一步

- 查看 [.claude/skills/tavily-web/SKILL.md](./.claude/skills/tavily-web/SKILL.md) 了解技术细节
- 开始提问，让 Claude 自动获取最新的网页信息！

## 故障排除

### 脚本执行失败
```bash
# 确保脚本有执行权限
chmod +x .claude/skills/tavily-web/tavily-api.cjs

# 确保 Node.js 已安装
node --version  # 应该显示版本号
```

### API 返回 401 错误
检查 API key 是否正确配置：
```bash
# 查看 .env 文件
cat .claude/skills/tavily-web/.env

# 确保格式正确
TAVILY_API_KEY=your_key_here  # ✅ 正确
TAVILY_API_KEY = your_key_here  # ❌ 错误（有空格）
```

### API 返回 429 错误
速率限制已达到，等待一段时间或升级 API key 配额。

---

🎉 配置完成！现在你可以享受强大的网页研究能力了！
