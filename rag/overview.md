# RAG 系统概述

> 检索增强生成（Retrieval-Augmented Generation）系统介绍

---

## 什么是 RAG？

**RAG**（Retrieval-Augmented Generation，检索增强生成）是一种结合了信息检索和生成式 AI 的技术架构。

### 核心思想

```
用户查询
  │
  ▼
┌─────────────┐
│  检索相关   │
│  文档/知识   │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  组合上下文  │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  LLM 生成   │
│  回答       │
└──────┬──────┘
       │
       ▼
   最终回答
```

---

## 为什么需要 RAG？

### 1. 解决知识过时问题

纯 LLM 的知识截止于训练数据，而 RAG 可以实时访问最新信息。

### 2. 减少幻觉

通过检索真实文档作为证据，减少 LLM 的虚假信息生成。

### 3. 领域知识集成

轻松集成企业内部知识库、产品文档等私有数据。

### 4. 可解释性

可以追溯回答来源，提供引用和证据。

---

## RAG 的核心组件

### 1. 文档加载（Document Loading）

从各种来源加载文档：
- PDF、Word、Markdown
- 网页
- 数据库
- API

### 2. 文档分块（Chunking）

将长文档分成适合检索的小块：
- 固定大小分块
- 语义分块
- 递归分块

### 3. 向量化（Embedding）

将文本转换为向量表示：
- 常用模型：text-embedding-3-small、jina-embeddings
- 向量维度：通常为 384-1536

### 4. 向量数据库（Vector Database）

存储和检索向量：
- OpenSearch
- Pinecone
- Chroma
- Weaviate

### 5. 检索（Retrieval）

根据查询找到最相关的文档：
- 向量相似度搜索
- 关键词搜索（BM25）
- 混合搜索

### 6. 生成（Generation）

LLM 根据检索到的上下文生成回答：
- Prompt Engineering
- 上下文限制
- 引用生成

---

## 生产级 RAG vs 玩具 RAG

| 特性 | 玩具 RAG | 生产级 RAG |
|------|---------|-----------|
| 搜索 | 仅向量搜索 | 混合搜索（向量 + 关键词） |
| 分块 | 固定大小 | 智能分块 |
| 监控 | 无 | Langfuse 追踪 |
| 缓存 | 无 | Redis 缓存 |
| 部署 | Streamlit | FastAPI + Docker |
| 错误处理 | 基本try-except | 重试、降级、监控 |
| 性能 | 未优化 | 连接池、批处理 |

---

## 学习路径

基于 [jamwithai/production-agentic-rag-course](https://github.com/jamwithai/production-agentic-rag-course) 的 7 周课程：

### Week 1: 基础设施搭建 ⭐
- Docker Compose 编排
- FastAPI 异步 API
- PostgreSQL 数据库
- OpenSearch 搜索引擎
- Apache Airflow 工作流
- Ollama 本地 LLM
- Redis 缓存
- Langfuse 可观测性

### Week 2: 数据摄取管道
- arXiv API 集成
- PDF 解析（Docling）
- Airflow DAG 编排
- 元数据提取

### Week 3: 搜索基础
- BM25 关键词搜索
- OpenSearch 索引设计
- 相关性评分

### Week 4: 智能分块和混合搜索
- 基于章节的分块
- 向量搜索集成
- RRF（Reciprocal Rank Fusion）
- 混合检索策略

### Week 5: 完整 RAG 系统
- 流式响应
- Gradio 界面
- 端到端集成

### Week 6: 生产监控和缓存
- Langfuse 追踪
- Redis 缓存策略
- 性能优化

### Week 7: Agentic RAG ⭐
- LangGraph 工作流
- 智能决策
- 文档评分
- 查询重写
- Telegram Bot 集成

---

## 参考资源

- [Production Agentic RAG Course](https://github.com/jamwithai/production-agentic-rag-course)
- [LangChain RAG 教程](https://python.langchain.com/docs/tutorials/rag/)
- [LlamaIndex 文档](https://docs.llamaindex.ai/)

---

**下一章**: [Week 1: 基础设施](week1-infrastructure.md)
