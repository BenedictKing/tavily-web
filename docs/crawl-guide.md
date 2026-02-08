# Crawl 爬取指南

## 概述

Tavily Crawl API 智能遍历网站，提取结构化内容。支持深度控制、路径过滤和语义聚焦。

---

## 核心概念

### Crawl vs Map

| 特性 | Crawl | Map |
|------|-------|-----|
| **返回内容** | 完整页面内容 | 仅 URL 列表 |
| **速度** | 较慢 | 快速 |
| **用途** | 内容提取 | URL 发现 |
| **成本** | 较高 | 较低 |

**选择建议：**
- 需要内容 → 使用 Crawl
- 仅需 URL → 使用 Map
- 大型站点 → 先 Map 后 Extract

---

## 核心参数

### url（必填）
起始 URL

```json
{"url": "https://docs.example.com"}
```

---

### instructions（推荐）
自然语言指令，语义聚焦爬取内容

```json
{
  "url": "https://docs.python.org",
  "instructions": "Find all pages about async/await and coroutines"
}
```

**优势：**
- 智能过滤相关页面
- 减少无关内容
- 提高爬取效率

---

### max_depth
爬取深度（1-5，默认 1）

```json
{
  "url": "https://example.com",
  "max_depth": 2
}
```

**说明：**
- `1`：仅起始页的直接链接
- `2`：二级链接
- `3+`：更深层次（谨慎使用）

**建议：**
- 小型站点：2-3
- 大型站点：1-2
- 特定路径：1 + 路径过滤

---

### max_breadth
每页最大链接数（默认 20）

```json
{
  "url": "https://example.com",
  "max_breadth": 50
}
```

**权衡：**
- 更大 = 更全面，但更慢
- 更小 = 更快，但可能遗漏

---

### limit
总页面数上限（默认 50）

```json
{
  "url": "https://example.com",
  "limit": 100
}
```

**建议：**
- 测试：10-20
- 中型站点：50-100
- 大型站点：100-500

---

## 7 个使用场景

### 1. 深层/未链接内容

**场景：** 内容未在主导航中链接

```json
{
  "url": "https://docs.example.com",
  "instructions": "Find all API reference pages",
  "max_depth": 3,
  "limit": 200
}
```

**适用于：**
- 文档站点
- 知识库
- 归档内容

---

### 2. 文档/结构化内容

**场景：** 提取完整文档集

```json
{
  "url": "https://docs.example.com",
  "instructions": "Extract all tutorial and guide pages",
  "select_paths": ["^/docs/.*", "^/guides/.*"],
  "format": "markdown",
  "limit": 100
}
```

**适用于：**
- API 文档
- 用户手册
- 教程网站

**后续处理：**
- 保存为本地 Markdown
- 构建 RAG 知识库
- 生成离线文档

---

### 3. 多模态/交叉引用

**场景：** 提取文本和图片

```json
{
  "url": "https://blog.example.com",
  "instructions": "Find all posts about machine learning",
  "include_images": true,
  "format": "markdown",
  "limit": 50
}
```

**适用于：**
- 博客文章
- 产品页面
- 新闻网站

---

### 4. 快速变化的内容

**场景：** 定期更新的内容

```json
{
  "url": "https://news.example.com",
  "instructions": "Latest articles from today",
  "max_depth": 1,
  "limit": 20
}
```

**适用于：**
- 新闻站点
- 论坛
- 社交媒体

**建议：**
- 定期运行（每小时/每天）
- 使用较小的 limit
- 结合时间戳过滤

---

### 5. RAG/知识库集成

**场景：** 构建向量数据库

```json
{
  "url": "https://docs.example.com",
  "instructions": "All documentation pages",
  "format": "markdown",
  "chunks_per_source": 5,
  "limit": 500
}
```

**流程：**
1. Crawl 提取内容
2. 分块（chunking）
3. 生成 embeddings
4. 存入向量数据库
5. 用于 RAG 查询

---

### 6. 合规/审计

**场景：** 内容审计和归档

```json
{
  "url": "https://company.com",
  "instructions": "All privacy policy and terms pages",
  "select_paths": ["^/legal/.*", "^/privacy/.*"],
  "format": "markdown",
  "limit": 50
}
```

**适用于：**
- 法律合规
- 内容归档
- 变更追踪

---

### 7. 已知 URL 模式

**场景：** 特定路径模式

```json
{
  "url": "https://api.example.com",
  "select_paths": ["^/api/v2/.*"],
  "exclude_paths": ["^/api/v1/.*"],
  "max_depth": 2,
  "limit": 100
}
```

**适用于：**
- API 文档特定版本
- 特定产品线
- 语言特定内容

---

## 路径过滤

### select_paths（包含）

```json
{
  "url": "https://docs.example.com",
  "select_paths": [
    "^/docs/api/.*",
    "^/docs/guides/.*"
  ]
}
```

**正则表达式示例：**
- `^/docs/.*` - 所有 /docs/ 下的页面
- `^/blog/2025/.*` - 2025 年的博客
- `.*\\.pdf$` - 所有 PDF 文件

---

### exclude_paths（排除）

```json
{
  "url": "https://example.com",
  "exclude_paths": [
    "^/admin/.*",
    "^/login.*",
    ".*\\?.*"  // 排除带查询参数的 URL
  ]
}
```

**常见排除：**
- 登录页面
- 管理后台
- 搜索结果页
- 分页链接

---

### 域名过滤

```json
{
  "url": "https://docs.example.com",
  "select_domains": ["docs\\.example\\.com"],
  "exclude_domains": ["ads\\.example\\.com"],
  "allow_external": false
}
```

---

## 高级功能

### 语义聚焦（instructions + chunks_per_source）

```json
{
  "url": "https://docs.example.com",
  "instructions": "Find pages about authentication and security",
  "chunks_per_source": 5,
  "extract_depth": "advanced"
}
```

**工作原理：**
1. 爬取所有页面
2. 根据 instructions 重排内容
3. 返回最相关的块

---

### 保存为文件

使用 `--output` 标志：

```bash
cat <<'JSON' | node tavily-api.cjs crawl --output results.json
{
  "url": "https://docs.example.com",
  "instructions": "All API documentation",
  "format": "markdown",
  "limit": 100
}
JSON
```

**后续处理：**
```javascript
const results = JSON.parse(fs.readFileSync('results.json'));
results.results.forEach(page => {
  const filename = page.url.replace(/[^a-z0-9]/gi, '_') + '.md';
  fs.writeFileSync(`./docs/${filename}`, page.raw_content);
});
```

---

## Map-then-Extract 模式

对于大型站点，先发现 URL，再选择性提取：

### 步骤 1：Map 发现 URL

```json
{
  "url": "https://docs.example.com",
  "instructions": "Find all API reference pages",
  "max_depth": 3,
  "limit": 1000
}
```

### 步骤 2：过滤 URL

```javascript
const relevantUrls = mapResults.urls.filter(url =>
  url.includes('/api/') && !url.includes('/deprecated/')
);
```

### 步骤 3：Extract 提取内容

```json
{
  "urls": relevantUrls.slice(0, 20),
  "format": "markdown"
}
```

**优势：**
- 更快的 URL 发现
- 精确控制提取内容
- 降低成本

---

## 性能优化

### 1. 合理设置 limit

```json
// ❌ 过大
{"limit": 10000}  // 可能超时或成本高

// ✅ 合理
{"limit": 100}    // 分批处理
```

---

### 2. 使用路径过滤

```json
// ❌ 无过滤
{"url": "https://example.com", "max_depth": 3}

// ✅ 有过滤
{
  "url": "https://example.com",
  "select_paths": ["^/docs/.*"],
  "max_depth": 2
}
```

---

### 3. 调整超时

```json
{
  "url": "https://slow-site.com",
  "timeout": 120  // 默认 150 秒
}
```

---

## 常见陷阱

### 1. 无限循环

**问题：** 动态生成的 URL

```json
// ❌ 可能陷入循环
{"url": "https://example.com/page?id=1"}

// ✅ 排除查询参数
{
  "url": "https://example.com",
  "exclude_paths": [".*\\?.*"]
}
```

---

### 2. 过度爬取

**问题：** 爬取整个站点

```json
// ❌ 过度
{"url": "https://wikipedia.org", "max_depth": 5}

// ✅ 限制
{
  "url": "https://wikipedia.org/wiki/Machine_learning",
  "max_depth": 1,
  "limit": 50
}
```

---

### 3. 忽略 robots.txt

**注意：** Tavily 遵守 robots.txt，某些页面可能无法爬取

**解决方案：**
- 检查站点的 robots.txt
- 使用 Extract API 直接提取已知 URL

---

## 完整示例

```json
{
  "url": "https://docs.python.org/3/",
  "instructions": "Find all pages about asyncio and concurrent programming",
  "max_depth": 2,
  "max_breadth": 30,
  "limit": 100,
  "select_paths": ["^/3/library/.*", "^/3/tutorial/.*"],
  "exclude_paths": ["^/3/whatsnew/.*"],
  "allow_external": false,
  "include_images": false,
  "extract_depth": "advanced",
  "format": "markdown",
  "timeout": 150
}
```

---

## 参考

- [Tavily Crawl API 文档](https://docs.tavily.com/api-reference/crawl)
- [Map API 文档](https://docs.tavily.com/api-reference/map)
- [正则表达式测试](https://regex101.com/)
