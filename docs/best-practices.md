# Tavily 最佳实践指南

## 概述

本指南提供使用 Tavily API 的最佳实践、性能优化技巧和常见模式。

---

## 端点选择决策树

```
需要什么？
│
├─ 搜索最新信息 → Search
│  ├─ 简单查询 → search_depth: "basic"
│  └─ 深度研究 → search_depth: "advanced"
│
├─ 提取已知 URL 内容 → Extract
│  ├─ 完整内容 → 不带 query
│  └─ 特定信息 → 带 query + chunks_per_source
│
├─ 遍历整个网站 → Crawl 或 Map
│  ├─ 需要内容 → Crawl
│  ├─ 仅需 URL → Map
│  └─ 大型站点 → Map + Extract
│
└─ AI 综合研究 → Research
   ├─ 简单问题 → model: "mini"
   └─ 复杂主题 → model: "pro"
```

---

## 查询优化

### Search 查询优化

#### ✅ 好的查询

```json
{
  "query": "Claude Opus 4 new features 2025",
  "search_depth": "basic",
  "max_results": 5
}
```

**特点：**
- 具体关键词
- <400 字符
- 明确意图

#### ❌ 不好的查询

```json
{
  "query": "Can you please tell me everything about the new Claude model that was released recently and what are all the features?",
  "search_depth": "advanced",
  "max_results": 20
}
```

**问题：**
- 过于冗长
- 自然语言描述
- 过度使用资源

---

### Extract 查询优化

#### ✅ 有效的 query

```json
{
  "urls": "https://docs.example.com/api",
  "query": "authentication methods OAuth2",
  "chunks_per_source": 3
}
```

#### ❌ 无效的 query

```json
{
  "urls": "https://docs.example.com/api",
  "query": "everything",
  "chunks_per_source": 5
}
```

---

## 性能优化

### 1. 控制结果数量

```json
// ❌ 过度
{
  "query": "AI news",
  "max_results": 20,
  "search_depth": "advanced"
}

// ✅ 合理
{
  "query": "AI news",
  "max_results": 5,
  "search_depth": "basic"
}
```

**建议：**
- 快速查询：3-5 个结果
- 标准搜索：5-10 个结果
- 深度研究：10-15 个结果

---

### 2. 选择合适的深度

| 场景 | 推荐深度 | 时间 |
|------|---------|------|
| 快速事实查询 | ultra-fast | <1s |
| 日常搜索 | basic | 5-10s |
| 深度研究 | advanced | 10-20s |

---

### 3. 使用域名过滤

```json
{
  "query": "tech news",
  "include_domains": ["techcrunch.com", "theverge.com"],
  "max_results": 5
}
```

**优势：**
- 提高结果质量
- 减少处理时间
- 降低成本

---

### 4. 合理使用 Crawl

```json
// ❌ 过度爬取
{
  "url": "https://wikipedia.org",
  "max_depth": 5,
  "limit": 10000
}

// ✅ 合理限制
{
  "url": "https://docs.example.com",
  "select_paths": ["^/api/.*"],
  "max_depth": 2,
  "limit": 100
}
```

---

## 成本优化

### 1. Map-then-Extract 模式

对于大型站点，先发现后提取：

```javascript
// 步骤 1：Map（低成本）
const mapResult = await tavilyMap({
  url: "https://docs.example.com",
  instructions: "Find API documentation pages",
  limit: 100
});

// 步骤 2：过滤 URL
const relevantUrls = mapResult.urls
  .filter(url => url.includes('/api/'))
  .slice(0, 20);

// 步骤 3：Extract（仅相关页面）
const content = await tavilyExtract({
  urls: relevantUrls,
  format: "markdown"
});
```

**节省：**
- 避免爬取无关页面
- 精确控制提取内容
- 降低 API 调用成本

---

### 2. 使用 include_raw_content 的时机

```json
// ❌ 不需要时也包含
{
  "query": "quick fact",
  "include_raw_content": true  // 浪费
}

// ✅ 仅在需要时包含
{
  "query": "detailed analysis",
  "include_raw_content": "markdown"  // 用于 RAG
}
```

---

### 3. 批量处理

```javascript
// ❌ 逐个处理
for (const url of urls) {
  await tavilyExtract({urls: url});
}

// ✅ 批量处理（最多 20 个）
const batches = chunk(urls, 20);
for (const batch of batches) {
  await tavilyExtract({urls: batch});
}
```

---

## RAG 集成模式

### 1. 搜索 + 提取模式

```javascript
// 步骤 1：搜索相关来源
const searchResult = await tavilySearch({
  query: "machine learning best practices",
  max_results: 5,
  include_raw_content: "markdown"
});

// 步骤 2：提取 raw_content
const documents = searchResult.results.map(r => ({
  content: r.raw_content,
  url: r.url,
  title: r.title
}));

// 步骤 3：分块和向量化
const chunks = documents.flatMap(doc =>
  chunkText(doc.content, 512)
);

// 步骤 4：存入向量数据库
await vectorDB.insert(chunks);
```

---

### 2. Crawl + RAG 模式

```javascript
// 步骤 1：爬取文档站点
const crawlResult = await tavilyCrawl({
  url: "https://docs.example.com",
  instructions: "All documentation pages",
  format: "markdown",
  limit: 200
});

// 步骤 2：处理每个页面
for (const page of crawlResult.results) {
  // 分块
  const chunks = chunkText(page.raw_content, 512);

  // 生成 embeddings
  const embeddings = await generateEmbeddings(chunks);

  // 存入向量数据库
  await vectorDB.insert({
    chunks,
    embeddings,
    metadata: {
      url: page.url,
      title: page.title
    }
  });
}

// 步骤 3：RAG 查询
async function ragQuery(question) {
  // 检索相关文档
  const relevant = await vectorDB.search(question, topK: 5);

  // 生成答案
  return await llm.generate({
    context: relevant.map(r => r.content).join('\n\n'),
    question
  });
}
```

---

### 3. 混合 RAG 模式

结合本地知识库和实时搜索：

```javascript
async function hybridRAG(question) {
  // 1. 搜索本地知识库
  const localResults = await vectorDB.search(question, topK: 3);

  // 2. 搜索最新信息
  const webResults = await tavilySearch({
    query: question,
    max_results: 3,
    include_raw_content: "markdown"
  });

  // 3. 合并上下文
  const context = [
    ...localResults.map(r => r.content),
    ...webResults.results.map(r => r.raw_content)
  ].join('\n\n');

  // 4. 生成答案
  return await llm.generate({context, question});
}
```

---

## 异步处理模式

### 1. 并行搜索

```javascript
const queries = [
  "AI news today",
  "quantum computing updates",
  "blockchain trends"
];

const results = await Promise.all(
  queries.map(q => tavilySearch({
    query: q,
    max_results: 5
  }))
);
```

---

### 2. 批量提取

```javascript
const urlGroups = chunk(allUrls, 20);

const results = await Promise.all(
  urlGroups.map(urls => tavilyExtract({
    urls,
    format: "markdown"
  }))
);
```

---

### 3. 流水线处理

```javascript
async function pipeline(startUrl) {
  // 1. Map 发现 URL
  const mapResult = await tavilyMap({
    url: startUrl,
    limit: 100
  });

  // 2. 过滤 URL
  const filtered = mapResult.urls.filter(isRelevant);

  // 3. 批量提取
  const batches = chunk(filtered, 20);
  const contents = [];

  for (const batch of batches) {
    const result = await tavilyExtract({urls: batch});
    contents.push(...result.results);
  }

  return contents;
}
```

---

## 错误处理

### 1. 重试机制

```javascript
async function tavilyWithRetry(fn, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await sleep(1000 * Math.pow(2, i)); // 指数退避
    }
  }
}

// 使用
const result = await tavilyWithRetry(() =>
  tavilySearch({query: "AI news"})
);
```

---

### 2. 超时处理

```javascript
async function tavilyWithTimeout(fn, timeout = 30000) {
  return Promise.race([
    fn(),
    new Promise((_, reject) =>
      setTimeout(() => reject(new Error('Timeout')), timeout)
    )
  ]);
}
```

---

### 3. 降级策略

```javascript
async function searchWithFallback(query) {
  try {
    // 尝试 advanced 搜索
    return await tavilySearch({
      query,
      search_depth: "advanced"
    });
  } catch (error) {
    // 降级到 basic
    return await tavilySearch({
      query,
      search_depth: "basic"
    });
  }
}
```

---

## 缓存策略

### 1. 结果缓存

```javascript
const cache = new Map();

async function cachedSearch(query, ttl = 3600000) {
  const key = `search:${query}`;
  const cached = cache.get(key);

  if (cached && Date.now() - cached.timestamp < ttl) {
    return cached.data;
  }

  const result = await tavilySearch({query});
  cache.set(key, {
    data: result,
    timestamp: Date.now()
  });

  return result;
}
```

---

### 2. URL 去重

```javascript
const processedUrls = new Set();

async function deduplicatedExtract(urls) {
  const newUrls = urls.filter(url => !processedUrls.has(url));

  if (newUrls.length === 0) return [];

  const result = await tavilyExtract({urls: newUrls});
  newUrls.forEach(url => processedUrls.add(url));

  return result;
}
```

---

## 监控和日志

### 1. 性能监控

```javascript
async function monitoredSearch(query) {
  const start = Date.now();

  try {
    const result = await tavilySearch({query});
    const duration = Date.now() - start;

    console.log(`Search completed in ${duration}ms`);
    return result;
  } catch (error) {
    console.error(`Search failed after ${Date.now() - start}ms:`, error);
    throw error;
  }
}
```

---

### 2. 使用统计

```javascript
const stats = {
  searches: 0,
  extracts: 0,
  crawls: 0,
  totalCost: 0
};

async function trackedSearch(query) {
  stats.searches++;
  const result = await tavilySearch({query});

  // 估算成本（示例）
  stats.totalCost += 0.01;

  return result;
}
```

---

## 常见陷阱

### 1. 过度爬取

```json
// ❌ 危险
{
  "url": "https://large-site.com",
  "max_depth": 5,
  "limit": 10000
}

// ✅ 安全
{
  "url": "https://large-site.com",
  "select_paths": ["^/docs/.*"],
  "max_depth": 2,
  "limit": 100
}
```

---

### 2. 忽略 robots.txt

**注意：** Tavily 遵守 robots.txt

**解决方案：**
- 检查目标站点的 robots.txt
- 使用其他工具（如 Firecrawl）处理受限站点

---

### 3. 未处理空结果

```javascript
// ❌ 未检查
const result = await tavilySearch({query});
const firstUrl = result.results[0].url; // 可能崩溃

// ✅ 安全
const result = await tavilySearch({query});
if (result.results && result.results.length > 0) {
  const firstUrl = result.results[0].url;
}
```

---

## 框架集成

### LangChain

```javascript
import { TavilySearchResults } from "@langchain/community/tools/tavily_search";

const tool = new TavilySearchResults({
  maxResults: 5,
  searchDepth: "advanced"
});

const result = await tool.invoke("AI news");
```

---

### LlamaIndex

```python
from llama_index.tools.tavily_research import TavilyToolSpec

tavily_tool = TavilyToolSpec(api_key="your-key")
agent = OpenAIAgent.from_tools(tavily_tool.to_tool_list())

response = agent.chat("Research AI trends")
```

---

### CrewAI

```python
from crewai import Agent, Task, Crew
from crewai_tools import TavilySearchResults

search_tool = TavilySearchResults()

researcher = Agent(
    role='Researcher',
    goal='Find latest AI news',
    tools=[search_tool]
)

task = Task(
    description='Research AI developments',
    agent=researcher
)

crew = Crew(agents=[researcher], tasks=[task])
result = crew.kickoff()
```

---

## 安全最佳实践

### 1. API 密钥管理

```bash
# ✅ 环境变量
export TAVILY_API_KEY=tvly-xxx

# ✅ .env 文件（不提交到 Git）
echo "TAVILY_API_KEY=tvly-xxx" > .env
echo ".env" >> .gitignore

# ❌ 硬编码
const apiKey = "tvly-xxx"; // 危险！
```

---

### 2. 输入验证

```javascript
function validateQuery(query) {
  if (!query || typeof query !== 'string') {
    throw new Error('Invalid query');
  }
  if (query.length > 400) {
    throw new Error('Query too long (max 400 chars)');
  }
  return query.trim();
}
```

---

### 3. 输出清理

```javascript
function sanitizeContent(content) {
  // 移除敏感信息
  return content
    .replace(/\b\d{3}-\d{2}-\d{4}\b/g, '[SSN]')
    .replace(/\b[\w.-]+@[\w.-]+\.\w+\b/g, '[EMAIL]');
}
```

---

## 参考

- [Tavily API 文档](https://docs.tavily.com/)
- [Search 指南](./search-guide.md)
- [Extract 指南](./extract-guide.md)
- [Crawl 指南](./crawl-guide.md)
- [Research 指南](./research-guide.md)
