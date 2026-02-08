# Search 搜索指南

## 概述

Tavily Search API 提供 LLM 优化的网页搜索，返回高质量、相关的结果。

---

## 核心参数

### query（必填）
搜索查询字符串

**最佳实践：**
- ✅ 保持在 400 字符以内
- ✅ 使用具体关键词
- ✅ 避免过于宽泛的查询
- ❌ 避免冗长的自然语言描述

```json
// ✅ 好的查询
{"query": "Claude Opus 4 release date features"}

// ❌ 不好的查询
{"query": "Can you please help me find information about when the new Claude Opus 4 model was released and what are all the new features and capabilities that it has compared to the previous version?"}
```

---

## 搜索深度选择

### ultra-fast
- **速度：** <1 秒
- **用途：** 快速事实查询
- **结果：** 基础相关性

### fast
- **速度：** 2-3 秒
- **用途：** 平衡性能和质量
- **结果：** 良好相关性

### basic（默认）
- **速度：** 5-10 秒
- **用途：** 标准搜索
- **结果：** 高质量结果

### advanced
- **速度：** 10-20 秒
- **用途：** 深度研究
- **结果：** 最全面的结果

```json
{
  "query": "quantum computing breakthroughs 2025",
  "search_depth": "advanced",
  "max_results": 10
}
```

---

## 使用场景

### 1. 实时新闻搜索

```json
{
  "query": "AI regulation news",
  "topic": "news",
  "time_range": "week",
  "max_results": 10,
  "include_answer": true
}
```

**适用于：**
- 新闻聚合
- 趋势分析
- 实时监控

---

### 2. 特定领域搜索

```json
{
  "query": "stock market analysis AAPL",
  "topic": "finance",
  "include_domains": ["bloomberg.com", "reuters.com"],
  "max_results": 5
}
```

**适用于：**
- 金融分析
- 行业研究
- 专业领域查询

---

### 3. 学术研究

```json
{
  "query": "machine learning interpretability methods",
  "search_depth": "advanced",
  "exclude_domains": ["wikipedia.org"],
  "include_raw_content": true,
  "max_results": 15
}
```

**适用于：**
- 文献综述
- 技术调研
- 深度分析

---

### 4. 本地化搜索

```json
{
  "query": "best restaurants",
  "country": "Japan",
  "max_results": 10
}
```

**适用于：**
- 地理特定查询
- 本地服务查找
- 区域信息

---

### 5. 时间范围搜索

```json
{
  "query": "AI breakthroughs",
  "start_date": "2025-01-01",
  "end_date": "2025-12-31",
  "max_results": 20
}
```

**适用于：**
- 历史事件
- 趋势分析
- 时间线研究

---

## 高级功能

### include_answer

获取 AI 生成的答案摘要：

```json
{
  "query": "what is quantum entanglement",
  "include_answer": "advanced",
  "max_results": 5
}
```

**选项：**
- `false`：不包含答案
- `true` / `"basic"`：基础答案
- `"advanced"`：详细答案

---

### include_raw_content

获取完整页面内容：

```json
{
  "query": "Python async tutorial",
  "include_raw_content": "markdown",
  "max_results": 3
}
```

**选项：**
- `false`：仅摘要
- `true` / `"text"`：纯文本
- `"markdown"`：Markdown 格式

**用途：**
- RAG 系统
- 内容分析
- 知识库构建

---

## 性能优化

### 1. 控制结果数量

```json
{
  "query": "AI news",
  "max_results": 5  // 默认 5，范围 1-20
}
```

**建议：**
- 快速查询：3-5 个结果
- 标准搜索：5-10 个结果
- 深度研究：10-20 个结果

---

### 2. 调整块数量

```json
{
  "query": "machine learning",
  "search_depth": "advanced",
  "chunks_per_source": 3  // 默认 3，范围 1-5
}
```

**说明：**
- 仅在 `advanced` 或 `fast` 深度下生效
- 更多块 = 更多上下文，但更慢

---

### 3. 域名过滤

```json
{
  "query": "tech news",
  "include_domains": ["techcrunch.com", "theverge.com"],
  "exclude_domains": ["spam-site.com"]
}
```

**优势：**
- 提高结果质量
- 减少噪音
- 加快搜索速度

---

## 常见模式

### 混合 RAG 模式

结合搜索和本地知识库：

```json
{
  "query": "company product features",
  "include_domains": ["yourcompany.com"],
  "include_raw_content": "markdown",
  "max_results": 10
}
```

**流程：**
1. 搜索获取最新信息
2. 提取 raw_content
3. 与本地知识库合并
4. 生成综合答案

---

### 异步搜索模式

对于多个查询：

```javascript
const queries = [
  "AI news today",
  "quantum computing updates",
  "blockchain trends"
];

const results = await Promise.all(
  queries.map(q => tavilySearch({query: q, max_results: 5}))
);
```

---

## 错误处理

### 查询过长

```json
// ❌ 错误
{"query": "very long query..." } // >400 字符

// ✅ 正确
{"query": "key terms only"}
```

---

### 无效的时间范围

```json
// ❌ 错误
{"time_range": "week", "start_date": "2025-01-01"}

// ✅ 正确：使用其中一个
{"time_range": "week"}
// 或
{"start_date": "2025-01-01", "end_date": "2025-12-31"}
```

---

## 完整示例

```json
{
  "query": "Claude AI capabilities 2025",
  "search_depth": "basic",
  "topic": "general",
  "max_results": 10,
  "chunks_per_source": 3,
  "time_range": "month",
  "include_answer": "advanced",
  "include_raw_content": "markdown",
  "include_images": false,
  "include_domains": ["anthropic.com", "techcrunch.com"],
  "exclude_domains": [],
  "country": null
}
```

---

## 参考

- [Tavily Search API 文档](https://docs.tavily.com/api-reference/search)
- [最佳实践](../README.md#best-practices)
