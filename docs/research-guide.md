# Research 研究指南

## 概述

Tavily Research API 提供 AI 驱动的深度研究，自动搜索、分析和综合信息，生成结构化报告。

---

## 核心参数

### input（必填）
研究主题或问题

```json
{
  "input": "What are the latest developments in quantum computing?"
}
```

**最佳实践：**
- 清晰具体的问题
- 避免过于宽泛
- 可以是复杂的研究主题

---

## 模型选择

### auto（默认）
自动根据查询复杂度选择模型

```json
{
  "input": "AI trends 2025",
  "model": "auto"
}
```

---

### mini
快速、针对性研究

```json
{
  "input": "What is the capital of France?",
  "model": "mini"
}
```

**特点：**
- 速度：快（10-30 秒）
- 适用：简单问题、事实查询
- 成本：低

---

### pro
全面、多代理研究

```json
{
  "input": "Comprehensive analysis of AI safety approaches",
  "model": "pro"
}
```

**特点：**
- 速度：慢（30-120 秒）
- 适用：复杂主题、深度分析
- 成本：高

**何时使用 pro：**
- 需要多角度分析
- 复杂的研究主题
- 需要全面的报告

---

## 使用场景

### 1. 市场研究

```json
{
  "input": "Current state of the electric vehicle market in 2025",
  "model": "pro",
  "citation_format": "numbered"
}
```

**适用于：**
- 行业分析
- 竞争情报
- 市场趋势

---

### 2. 技术调研

```json
{
  "input": "Comparison of modern web frameworks: React, Vue, and Svelte",
  "model": "pro",
  "citation_format": "numbered"
}
```

**适用于：**
- 技术选型
- 架构决策
- 工具评估

---

### 3. 学术研究

```json
{
  "input": "Recent advances in natural language processing transformers",
  "model": "pro",
  "citation_format": "apa"
}
```

**适用于：**
- 文献综述
- 研究背景
- 论文写作

---

### 4. 商业分析

```json
{
  "input": "Financial performance and growth strategy of Tesla in 2024-2025",
  "model": "pro",
  "output_schema": {
    "type": "object",
    "properties": {
      "revenue": {"type": "number"},
      "growth_rate": {"type": "number"},
      "key_strategies": {
        "type": "array",
        "items": {"type": "string"}
      }
    }
  }
}
```

**适用于：**
- 投资分析
- 尽职调查
- 战略规划

---

### 5. 快速事实查询

```json
{
  "input": "Who won the 2024 Nobel Prize in Physics?",
  "model": "mini",
  "citation_format": "numbered"
}
```

**适用于：**
- 事实核查
- 快速查询
- 简单问题

---

## 结构化输出

### output_schema

使用 JSON Schema 定义输出结构：

```json
{
  "input": "Analyze Apple's Q4 2024 earnings",
  "model": "pro",
  "output_schema": {
    "type": "object",
    "properties": {
      "company": {
        "type": "string",
        "description": "Company name"
      },
      "quarter": {
        "type": "string",
        "description": "Fiscal quarter"
      },
      "revenue": {
        "type": "number",
        "description": "Total revenue in billions"
      },
      "key_metrics": {
        "type": "array",
        "description": "Key performance indicators",
        "items": {"type": "string"}
      },
      "outlook": {
        "type": "string",
        "description": "Future outlook summary"
      }
    },
    "required": ["company", "revenue"]
  }
}
```

**返回示例：**
```json
{
  "company": "Apple Inc.",
  "quarter": "Q4 2024",
  "revenue": 123.5,
  "key_metrics": [
    "iPhone sales up 15%",
    "Services revenue record high",
    "Strong growth in emerging markets"
  ],
  "outlook": "Positive growth expected in 2025"
}
```

---

### 复杂嵌套结构

```json
{
  "input": "Compare top 3 AI companies",
  "model": "pro",
  "output_schema": {
    "type": "object",
    "properties": {
      "companies": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "name": {"type": "string"},
            "valuation": {"type": "number"},
            "key_products": {
              "type": "array",
              "items": {"type": "string"}
            },
            "strengths": {"type": "string"},
            "weaknesses": {"type": "string"}
          }
        }
      }
    }
  }
}
```

---

## 引用格式

### numbered（默认）
数字引用

```json
{
  "input": "AI safety research",
  "citation_format": "numbered"
}
```

**输出：**
```
AI safety is a critical concern [1]. Recent research shows... [2].

References:
[1] https://example.com/source1
[2] https://example.com/source2
```

---

### apa
APA 格式

```json
{
  "input": "Climate change impacts",
  "citation_format": "apa"
}
```

**输出：**
```
Climate change affects... (Smith, 2024).

References:
Smith, J. (2024). Climate impacts. Nature.
```

---

### mla
MLA 格式

```json
{
  "input": "Shakespeare's influence",
  "citation_format": "mla"
}
```

---

### chicago
Chicago 格式

```json
{
  "input": "Historical analysis",
  "citation_format": "chicago"
}
```

---

## 流式输出（不推荐）

```json
{
  "input": "AI trends",
  "stream": false  // 推荐设为 false
}
```

**说明：**
- 流式输出会增加 token 消耗
- 在 Claude Code 中建议禁用
- 官方脚本默认禁用

---

## 保存结果

使用 `--output` 标志：

```bash
cat <<'JSON' | node tavily-api.cjs research --output report.md
{
  "input": "Comprehensive analysis of AI agent frameworks",
  "model": "pro",
  "citation_format": "numbered"
}
JSON
```

**后续处理：**
```bash
# 查看报告
cat report.md

# 转换为 PDF
pandoc report.md -o report.pdf
```

---

## 性能优化

### 1. 选择合适的模型

```json
// ❌ 过度使用 pro
{
  "input": "What is 2+2?",
  "model": "pro"  // 浪费
}

// ✅ 合理选择
{
  "input": "What is 2+2?",
  "model": "mini"  // 或 auto
}
```

---

### 2. 明确研究范围

```json
// ❌ 过于宽泛
{
  "input": "Tell me everything about AI"
}

// ✅ 具体明确
{
  "input": "Latest developments in large language models in 2025"
}
```

---

### 3. 使用结构化输出

```json
// ✅ 结构化
{
  "input": "AI companies analysis",
  "output_schema": {...}  // 明确结构
}
```

**优势：**
- 易于解析
- 一致的格式
- 减少后处理

---

## 与其他端点的对比

| 特性 | Research | Search | Crawl |
|------|----------|--------|-------|
| **输入** | 研究主题 | 搜索查询 | 起始 URL |
| **输出** | AI 综合报告 | 搜索结果 | 页面内容 |
| **引用** | 自动生成 | 可选 | 无 |
| **结构化** | 支持 schema | 不支持 | 不支持 |
| **速度** | 慢（30-120s） | 快（5-10s） | 中等 |
| **用途** | 深度研究 | 快速查找 | 内容提取 |

---

## 常见模式

### 研究 + 验证

```javascript
// 步骤 1：Research 生成报告
const report = await tavilyResearch({
  input: "AI safety best practices",
  model: "pro"
});

// 步骤 2：Search 验证关键点
const verification = await tavilySearch({
  query: "AI safety alignment techniques",
  max_results: 5
});
```

---

### 结构化数据提取

```javascript
const companies = await tavilyResearch({
  input: "Top 5 AI startups in 2025",
  model: "pro",
  output_schema: {
    type: "object",
    properties: {
      companies: {
        type: "array",
        items: {
          type: "object",
          properties: {
            name: {type: "string"},
            funding: {type: "number"},
            focus: {type: "string"}
          }
        }
      }
    }
  }
});

// 直接使用结构化数据
companies.companies.forEach(c => {
  console.log(`${c.name}: $${c.funding}M - ${c.focus}`);
});
```

---

## 错误处理

### 超时

```json
// 如果研究超时
{
  "input": "Very complex topic...",
  "model": "pro"
}
```

**解决方案：**
- 缩小研究范围
- 使用 `model: "mini"`
- 分解为多个小问题

---

### 无相关结果

```json
// 如果主题过于小众
{
  "input": "Obscure topic with no online info"
}
```

**解决方案：**
- 调整查询措辞
- 使用更通用的术语
- 尝试相关主题

---

## 完整示例

```json
{
  "input": "Comprehensive analysis of the impact of AI on software development in 2025",
  "model": "pro",
  "stream": false,
  "citation_format": "numbered",
  "output_schema": {
    "type": "object",
    "properties": {
      "summary": {
        "type": "string",
        "description": "Executive summary"
      },
      "key_trends": {
        "type": "array",
        "description": "Major trends identified",
        "items": {"type": "string"}
      },
      "tools": {
        "type": "array",
        "description": "Popular AI tools for developers",
        "items": {
          "type": "object",
          "properties": {
            "name": {"type": "string"},
            "category": {"type": "string"},
            "adoption_rate": {"type": "string"}
          }
        }
      },
      "challenges": {
        "type": "array",
        "description": "Key challenges",
        "items": {"type": "string"}
      },
      "future_outlook": {
        "type": "string",
        "description": "Predictions for the future"
      }
    },
    "required": ["summary", "key_trends"]
  }
}
```

---

## 参考

- [Tavily Research API 文档](https://docs.tavily.com/api-reference/research)
- [JSON Schema 规范](https://json-schema.org/)
- [引用格式指南](https://www.citationmachine.net/)
