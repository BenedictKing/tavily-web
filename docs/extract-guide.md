# Extract 提取指南

## 概述

Tavily Extract API 从指定 URL 提取干净、结构化的内容。支持单步提取和查询重排。

---

## 核心参数

### urls（必填）
单个 URL 或 URL 数组（最多 20 个）

```json
// 单个 URL
{"urls": "https://example.com"}

// 多个 URL
{"urls": ["https://example.com/page1", "https://example.com/page2"]}
```

---

## 两种提取模式

### 1. 单步提取（无 query）

直接提取完整内容：

```json
{
  "urls": "https://docs.example.com/api",
  "extract_depth": "basic",
  "format": "markdown"
}
```

**适用于：**
- 已知相关 URL
- 需要完整内容
- 简单页面

---

### 2. 两步提取（带 query）

使用查询重排内容块：

```json
{
  "urls": "https://docs.example.com/api",
  "query": "authentication methods",
  "chunks_per_source": 3,
  "extract_depth": "basic"
}
```

**工作原理：**
1. 提取完整内容
2. 分块（chunking）
3. 根据 query 重排
4. 返回最相关的块

**适用于：**
- 长文档
- 需要特定信息
- 减少 token 消耗

---

## 使用场景

### 1. 文档提取

```json
{
  "urls": [
    "https://docs.python.org/3/library/asyncio.html",
    "https://docs.python.org/3/library/concurrent.html"
  ],
  "format": "markdown",
  "extract_depth": "basic"
}
```

**适用于：**
- API 文档
- 技术手册
- 教程页面

---

### 2. 新闻文章提取

```json
{
  "urls": "https://news.example.com/article",
  "query": "key findings and conclusions",
  "chunks_per_source": 3,
  "format": "markdown"
}
```

**适用于：**
- 新闻摘要
- 文章分析
- 内容聚合

---

### 3. 产品页面提取

```json
{
  "urls": "https://shop.example.com/product/123",
  "query": "price, specifications, reviews",
  "chunks_per_source": 5,
  "include_images": true
}
```

**适用于：**
- 价格监控
- 产品比较
- 评论分析

---

### 4. 学术论文提取

```json
{
  "urls": "https://arxiv.org/abs/2301.12345",
  "query": "methodology and results",
  "chunks_per_source": 5,
  "extract_depth": "advanced",
  "format": "markdown"
}
```

**适用于：**
- 文献综述
- 研究分析
- 引用提取

---

### 5. 批量 URL 提取

```json
{
  "urls": [
    "https://blog.example.com/post1",
    "https://blog.example.com/post2",
    "https://blog.example.com/post3"
  ],
  "query": "main topics and key points",
  "chunks_per_source": 3
}
```

**适用于：**
- 内容聚合
- 批量分析
- 知识库构建

---

## extract_depth 选择

### basic（默认）
- **速度：** 快
- **适用：** 静态 HTML 页面
- **成本：** 低

```json
{
  "urls": "https://example.com",
  "extract_depth": "basic"
}
```

---

### advanced
- **速度：** 慢
- **适用：** JavaScript 渲染页面、复杂页面
- **成本：** 高

```json
{
  "urls": "https://spa-app.example.com",
  "extract_depth": "advanced"
}
```

**何时使用 advanced：**
- 单页应用（SPA）
- 动态加载内容
- 需要 JavaScript 执行
- basic 模式提取失败

---

## format 选择

### markdown（默认）
保留格式的 Markdown：

```json
{
  "urls": "https://example.com",
  "format": "markdown"
}
```

**优势：**
- 保留结构
- 易于阅读
- 适合 RAG

---

### text
纯文本：

```json
{
  "urls": "https://example.com",
  "format": "text"
}
```

**优势：**
- 更小的体积
- 简单处理
- 适合分析

---

## 查询重排优化

### chunks_per_source

控制返回的块数量（1-5，默认 3）：

```json
{
  "urls": "https://long-article.com",
  "query": "specific topic",
  "chunks_per_source": 5
}
```

**建议：**
- 短文档：1-2 块
- 中等文档：3 块
- 长文档：4-5 块

---

### 编写有效的 query

```json
// ✅ 好的 query
{
  "urls": "https://docs.example.com",
  "query": "installation steps and requirements"
}

// ❌ 不好的 query
{
  "urls": "https://docs.example.com",
  "query": "everything"
}
```

**最佳实践：**
- 具体明确
- 使用关键词
- 避免过于宽泛

---

## 高级功能

### 包含图片

```json
{
  "urls": "https://blog.example.com/post",
  "include_images": true,
  "format": "markdown"
}
```

**返回：**
- 图片 URL
- 图片描述（如果有）
- Markdown 格式的图片引用

---

### 超时控制

```json
{
  "urls": "https://slow-site.com",
  "timeout": 30.0  // 秒
}
```

**建议：**
- 快速站点：10-20 秒
- 慢速站点：30-60 秒

---

## 与 Crawl 的对比

| 特性 | Extract | Crawl |
|------|---------|-------|
| **输入** | 已知 URL | 起始 URL |
| **发现** | 不发现新 URL | 自动发现链接 |
| **速度** | 快 | 慢 |
| **用途** | 特定页面 | 整个站点 |
| **成本** | 低 | 高 |

**选择建议：**
- 已知 URL → Extract
- 需要发现 URL → Crawl
- 大量 URL → Map + Extract

---

## Map-then-Extract 模式

对于大型站点，先发现后提取：

### 步骤 1：Map 发现 URL

```json
{
  "url": "https://docs.example.com",
  "instructions": "Find all API reference pages",
  "limit": 100
}
```

### 步骤 2：Extract 提取内容

```json
{
  "urls": [
    "https://docs.example.com/api/auth",
    "https://docs.example.com/api/users",
    "https://docs.example.com/api/data"
  ],
  "query": "request parameters and response format",
  "chunks_per_source": 3
}
```

**优势：**
- 精确控制
- 降低成本
- 更快的处理

---

## 批量处理模式

### 分批提取

```javascript
const urls = [...]; // 100 个 URL
const batchSize = 20;

for (let i = 0; i < urls.length; i += batchSize) {
  const batch = urls.slice(i, i + batchSize);
  const result = await tavilyExtract({
    urls: batch,
    format: "markdown"
  });
  // 处理结果
}
```

---

### 并行提取

```javascript
const urlGroups = [
  ["url1", "url2"],
  ["url3", "url4"],
  ["url5", "url6"]
];

const results = await Promise.all(
  urlGroups.map(urls => tavilyExtract({urls}))
);
```

---

## 错误处理

### URL 无法访问

```json
// 返回
{
  "results": [
    {
      "url": "https://example.com",
      "error": "Failed to fetch"
    }
  ]
}
```

**解决方案：**
- 检查 URL 是否有效
- 尝试 `extract_depth: "advanced"`
- 增加 timeout

---

### 内容为空

```json
// 返回
{
  "results": [
    {
      "url": "https://example.com",
      "raw_content": ""
    }
  ]
}
```

**可能原因：**
- 页面需要 JavaScript
- 内容被 robots.txt 阻止
- 页面需要登录

**解决方案：**
- 使用 `extract_depth: "advanced"`
- 检查 robots.txt
- 使用其他工具（如 Firecrawl）

---

## 完整示例

```json
{
  "urls": [
    "https://docs.example.com/api/authentication",
    "https://docs.example.com/api/authorization"
  ],
  "query": "OAuth2 flow and token management",
  "chunks_per_source": 4,
  "extract_depth": "basic",
  "format": "markdown",
  "include_images": false,
  "include_favicon": false,
  "timeout": 30.0
}
```

---

## 参考

- [Tavily Extract API 文档](https://docs.tavily.com/api-reference/extract)
- [Crawl vs Extract 对比](./crawl-guide.md#crawl-vs-map)
- [Map API 文档](https://docs.tavily.com/api-reference/map)
