* [首页](/)
* [关于本项目](README.md)

## 1. 基础篇

### 1.1 AI 基础概念
* [AI 基础概念](basics/ai-concepts.md)
* [机器学习入门](basics/machine-learning.md)
* [深度学习基础](basics/deep-learning.md)

## 2. Agentic RAG 系统

### 2.1 RAG 概述
* [RAG 系统概述](agentic-rag/overview.md)

### 2.2 基础设施搭建
* [Week 1: 基础设施搭建](agentic-rag/week1-infrastructure.md)
  * [Docker & Docker Compose](agentic-rag/week1-infrastructure.md#1--docker--docker-compose)
  * [FastAPI](agentic-rag/week1-infrastructure.md#2--fastapi)
  * [PostgreSQL](agentic-rag/week1-infrastructure.md#3--postgresql)
  * [OpenSearch](agentic-rag/week1-infrastructure.md#4--opensearch)
  * [Apache Airflow](agentic-rag/week1-infrastructure.md#5--apache-airflow)
  * [Ollama](agentic-rag/week1-infrastructure.md#6--ollama)
  * [Redis](agentic-rag/week1-infrastructure.md#7--redis)
  * [Langfuse](agentic-rag/week1-infrastructure.md#8--langfuse)

### 2.3 数据管道（待添加）
* [Week 2: 数据摄取管道](agentic-rag/week2-data-pipeline.md)
  * [arXiv API 集成](agentic-rag/week2-data-pipeline.md#1--arxiv-api-集成)
  * [PDF 解析](agentic-rag/week2-data-pipeline.md#2--pdf-解析)
  * [Airflow DAG 编排](agentic-rag/week2-data-pipeline.md#3--airflow-dag-编排)

### 2.4 搜索系统（待添加）
* [Week 3: 关键词搜索基础](agentic-rag/week3-search-basics.md)
  * [BM25 算法](agentic-rag/week3-search-basics.md#1--bm25-算法)
  * [OpenSearch 索引设计](agentic-rag/week3-search-basics.md#2--opensearch-索引设计)
  * [相关性评分](agentic-rag/week3-search-basics.md#3--相关性评分)

### 2.5 混合搜索（待添加）
* [Week 4: 混合搜索实现](agentic-rag/week4-hybrid-search.md)
  * [智能分块策略](agentic-rag/week4-hybrid-search.md#1--智能分块策略)
  * [向量搜索集成](agentic-rag/week4-hybrid-search.md#2--向量搜索集成)
  * [RRF 融合算法](agentic-rag/week4-hybrid-search.md#3--rrf-融合算法)

### 2.6 完整 RAG 系统（待添加）
* [Week 5: 端到端 RAG 实现](agentic-rag/week5-complete-rag.md)
  * [流式响应](agentic-rag/week5-complete-rag.md#1--流式响应)
  * [Gradio 界面](agentic-rag/week5-complete-rag.md#2--gradio-界面)
  * [端到端集成](agentic-rag/week5-complete-rag.md#3--端到端集成)

### 2.7 生产监控与缓存（待添加）
* [Week 6: 生产级优化](agentic-rag/week6-monitoring.md)
  * [Langfuse 追踪](agentic-rag/week6-monitoring.md#1--langfuse-追踪)
  * [Redis 缓存策略](agentic-rag/week6-monitoring.md#2--redis-缓存策略)
  * [性能优化](agentic-rag/week6-monitoring.md#3--性能优化)

### 2.8 Agentic RAG（待添加）
* [Week 7: Agentic AI 与 LangGraph](agentic-rag/week7-agentic-rag.md) ⭐
  * [LangGraph 工作流](agentic-rag/week7-agentic-rag.md#1--langgraph-工作流)
  * [智能决策机制](agentic-rag/week7-agentic-rag.md#2--智能决策机制)
  * [文档评分系统](agentic-rag/week7-agentic-rag.md#3--文档评分系统)
  * [查询重写](agentic-rag/week7-agentic-rag.md#4--查询重写)
  * [Telegram Bot 集成](agentic-rag/week7-agentic-rag.md#5--telegram-bot-集成)

## 3. LLM 应用开发

* [LLM 基础](llm/basics.md)
* [Prompt Engineering](llm/prompt-engineering.md)
* [Agent 开发](llm/agent-development.md)

## 4. 实践项目

* [生产级 RAG 系统](projects/production-rag.md)
* [OpenClaw 学习笔记](projects/openclaw-learning.md)
* [实战案例分析](projects/case-studies.md)

## 5. 附录

* [术语表](appendix/glossary.md)
* [常见问题](appendix/faq.md)
* [参考资源](appendix/resources.md)
