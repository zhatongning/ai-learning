# Week 1: Infrastructure Foundation - 详细学习指南

> 基于 jamwithai/production-agentic-rag-course 第一周内容整理

---

## 📚 目录

1. [项目概述](#项目概述)
2. [核心理念：基础设施优先](#核心理念基础设施优先)
3. [技术组件详解](#技术组件详解)
4. [Docker Compose 架构](#docker-compose-架构)
5. [实践指南](#实践指南)
6. [配置详解](#配置详解)
7. [常见问题与解决方案](#常见问题与解决方案)
8. [学习检查清单](#学习检查清单)
9. [延伸阅读](#延伸阅读)

---

## 项目概述

### 🎯 What We're Building

**arXiv Paper Curator** - 一个生产级的 RAG (Retrieval-Augmented Generation) 系统，用于：

- 自动从 arXiv 获取学术论文
- 智能解析和处理 PDF 文档
- 使用混合搜索（关键词 + 语义）检索相关论文
- 基于 LLM 回答研究问题
- 完整的生产级监控和可观测性

### 📊 项目统计

- **Stars**: 4.8k+
- **Forks**: 1.2k+
- **GitHub**: https://github.com/jamwithai/production-agentic-rag-course
- **博客**: https://jamwithai.substack.com/p/the-infrastructure-that-powers-rag

### 🚀 学习目标

**Week 1 专注于基础设施搭建，你将学会：**

✅ Docker Compose 编排管理多个服务  
✅ FastAPI 构建异步 API  
✅ PostgreSQL 数据库设计与优化  
✅ OpenSearch 混合搜索引擎配置  
✅ Apache Airflow 工作流编排  
✅ Ollama 本地 LLM 服务  
✅ Redis 缓存配置  
✅ Langfuse 可观测性集成  
✅ 健康检查和服务发现  

---

## 核心理念：基础设施优先

### ⚠️ 为什么 85% 的 AI 项目失败？

不是因为模型不好，而是因为**基础设施有问题**。

### 🎓 传统 RAG 教程的问题

```
传统教程的路径：
1. 直接上手向量搜索
2. 使用玩具数据集
3. 部署到 Streamlit 就说是"生产环境"
4. 只关注准确率指标
```

### ✅ 生产级方法

```
我们的路径：
1. 构建稳固的基础设施 ✅ (Week 1)
2. 实现数据摄取管道
3. 实现关键词搜索基础
4. 添加语义层
5. 完整的 RAG 管道
6. 生产监控和优化
7. Agentic RAG
```

### 💡 关键洞察

**基础设施是第一公民，不是事后诸葛亮**

#### 为什么基础设施优先？

1. **容错性**: 系统不会因为负载而崩溃
2. **可维护性**: 团队可以维护和扩展代码
3. **可观测性**: 监控告诉你系统实际发生了什么
4. **自动化**: 管道无需人工干预就能运行

#### 生产级思维模式

| 维度 | 开发思维 | 生产思维 |
|------|---------|---------|
| 关注点 | 功能实现 | 稳定性、性能、成本 |
| 错误处理 | try-except | 重试、降级、监控 |
| 数据 | 内存中 | 持久化、备份 |
| 部署 | 手动运行 | 自动化、CI/CD |
| 监控 | print 调试 | 指标、日志、追踪 |

---

## 技术组件详解

### 1. 🐳 Docker & Docker Compose

#### 作用
- **容器化**: 将应用和依赖打包成独立的容器
- **编排**: 管理多个容器的启动、停止、网络和存储
- **一致性**: 确保开发、测试、生产环境一致

#### 为什么选择 Docker？

1. **隔离性**: 每个服务独立运行，避免依赖冲突
2. **可移植性**: 一次构建，到处运行
3. **可扩展性**: 易于水平扩展服务
4. **标准化**: 团队成员使用相同的环境

#### 架构图

```
┌─────────────────────────────────────────────────┐
│            Docker Compose 网络                 │
│           (rag-network bridge)                 │
└─────────────────────────────────────────────────┘
                      │
        ┌─────────────┼─────────────┐
        │             │             │
   ┌────▼────┐  ┌──▼───┐  ┌────▼────┐
   │  API    │  │ Open │  │ Postgres│
   │ Service │  │Search│  │  DB    │
   └─────────┘  └──────┘  └─────────┘
   ┌─────────┐  ┌──────┐  ┌─────────┐
   │ Airflow │  │ Ollama│  │  Redis  │
   └─────────┘  └──────┘  └─────────┘
   ┌─────────┐  ┌──────┐
   │Langfuse │  │Gradio│
   └─────────┘  └──────┘
```

---

### 2. ⚡ FastAPI

#### 作用
- **异步 Web 框架**: 构建高性能的 REST API
- **自动文档**: 自动生成 OpenAPI/Swagger 文档
- **类型验证**: 使用 Pydantic 进行请求/响应验证

#### 核心特性

```python
# FastAPI 关键模式
from fastapi import FastAPI, Depends
from sqlalchemy.orm import Session

app = FastAPI(
    title="ArXiv Paper Curator API",
    description="Production-grade RAG system for academic papers",
    version="1.0.0"
)

# 依赖注入模式
def get_db():
    """数据库会话依赖"""
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

# 异步端点
@app.get("/api/v1/papers")
async def list_papers(
    skip: int = 0,
    limit: int = 10,
    db: Session = Depends(get_db)
):
    """异步获取论文列表"""
    papers = paper_repository.get_all(db, skip=skip, limit=limit)
    return papers

# 健康检查端点
@app.get("/api/v1/health")
async def health_check():
    """服务健康状态检查"""
    return {
        "status": "healthy",
        "services": {
            "database": check_database(),
            "opensearch": check_opensearch(),
            "redis": check_redis()
        }
    }
```

#### 为什么选择 FastAPI？

| 特性 | FastAPI | Flask | Django |
|------|---------|-------|--------|
| 性能 | ⚡⚡⚡ | ⚡⚡ | ⚡ |
| 异步支持 | ✅ 原生 | ⚠️ 扩展 | ⚠️ 扩展 |
| 自动文档 | ✅ Swagger | ⚠️ 手动 | ⚠️ 手动 |
| 类型提示 | ✅ Pydantic | ⚠️ 可选 | ⚠️ 可选 |
| 学习曲线 | 📈 中等 | 📉 简单 | 📈 陡峭 |

#### 生产级最佳实践

1. **依赖注入**: 用于数据库会话、认证、配置
2. **中间件**: 错误处理、日志、CORS
3. **异步操作**: 提高并发性能
4. **健康检查**: 用于负载均衡和服务发现
5. **分页**: 防止大量数据返回

---

### 3. 🗄️ PostgreSQL

#### 作用
- **关系型数据库**: 存储结构化数据
- **元数据管理**: 存储论文的作者、标题、摘要等
- **事务支持**: 确保数据一致性

#### 架构设计

```sql
-- 论文表结构（示例）
CREATE TABLE papers (
    id SERIAL PRIMARY KEY,
    arxiv_id VARCHAR(50) UNIQUE NOT NULL,
    title TEXT NOT NULL,
    authors JSONB,  -- 存储作者数组
    abstract TEXT,
    published_date DATE,
    categories TEXT[],
    pdf_url TEXT,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- JSONB 索引（用于快速查询）
CREATE INDEX idx_papers_authors ON papers USING GIN (authors);
CREATE INDEX idx_papers_categories ON papers USING GIN (categories);

-- 复合索引（用于组合查询）
CREATE INDEX idx_papers_date_category 
ON papers(published_date DESC, categories);
```

#### 配置参数

```yaml
# Docker Compose 配置
postgres:
  image: postgres:16-alpine
  container_name: rag-postgres
  environment:
    POSTGRES_DB: rag_db
    POSTGRES_USER: rag_user
    POSTGRES_PASSWORD: rag_password
    POSTGRES_HOST_AUTH_METHOD: password
    PGDATA: /var/lib/postgresql/data/pgdata  # 数据目录
  ports:
    - "5432:5432"
  volumes:
    - postgres_data:/var/lib/postgresql/data  # 持久化
  healthcheck:
    test: ["CMD-SHELL", "pg_isready -U rag_user -d rag_db"]
    interval: 5s
    timeout: 5s
    retries: 10
    start_period: 30s
  restart: unless-stopped
```

#### 为什么选择 PostgreSQL？

1. **JSONB 支持**: 灵活存储半结构化数据
2. **全文搜索**: 内置的文本搜索功能
3. **扩展性**: 支持 PostGIS、pgvector 等扩展
4. **可靠性**: ACID 事务支持
5. **性能**: 高效的查询优化器

#### 连接池管理

```python
# 使用 SQLAlchemy 管理连接池
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

# 连接池配置
engine = create_engine(
    DATABASE_URL,
    pool_size=10,          # 连接池大小
    max_overflow=20,       # 最大溢出连接数
    pool_timeout=30,       # 获取连接超时时间
    pool_recycle=3600,     # 连接回收时间（1小时）
    pool_pre_ping=True      # 连接前检测连接是否有效
)

SessionLocal = sessionmaker(bind=engine)
Base = declarative_base()
```

---

### 4. 🔍 OpenSearch

#### 作用
- **搜索引擎**: 全文搜索和结构化搜索
- **混合搜索**: 结合 BM25 关键词搜索和向量搜索
- **可扩展**: 分布式架构，支持水平扩展

#### 核心概念

##### 1. BM25 算法
BM25 是一种用于信息检索的概率排序函数，它基于 TF-IDF 并添加了文档长度归一化。

```
BM25(D,Q) = Σ IDF(qi) * (f(qi,D) * (k1 + 1)) / 
           (f(qi,D) + k1 * (1 - b + b * |D|/avgdl))

其中:
- D: 文档
- Q: 查询
- qi: 查询中的第 i 个词
- f(qi,D): 词 qi 在文档 D 中的词频
- |D|: 文档长度
- avgdl: 平均文档长度
- k1, b: 可调参数 (k1 通常在 1.2-2.0, b 在 0.75)
- IDF: 逆文档频率
```

##### 2. 向量搜索
使用嵌入模型将文本转换为向量，然后计算余弦相似度。

```python
# 向量搜索示例
from openai import OpenAI

client = OpenAI(api_key="your-api-key")

# 生成嵌入
embedding = client.embeddings.create(
    input="What is attention mechanism?",
    model="text-embedding-3-small"
)

vector = embedding.data[0].embedding  # 1024 维向量
```

##### 3. 混合搜索 (Hybrid Search)
结合 BM25 和向量搜索的优势：

```json
{
  "query": {
    "bool": {
      "should": [
        {
          "multi_match": {
            "query": "attention mechanism transformer",
            "fields": ["title^2", "abstract", "content"],
            "type": "best_fields"
          }
        },
        {
          "knn": {
            "vector_embedding": {
              "vector": [0.1, 0.2, ...],
              "k": 10
            }
          }
        }
      ]
    }
  }
}
```

##### 4. RRF (Reciprocal Rank Fusion)
融合多个搜索结果的算法：

```
RRF(d) = Σ 1 / (k + rank_i(d))

其中:
- d: 文档
- rank_i(d): 文档在第 i 个搜索中的排名
- k: 常数 (通常为 60)
```

#### 配置示例

```yaml
# OpenSearch 单节点配置
opensearch:
  image: opensearchproject/opensearch:2.19.0
  container_name: rag-opensearch
  environment:
    discovery.type: single-node
    OPENSEARCH_JAVA_OPTS: "-Xms512m -Xmx512m"  # JVM 内存
    DISABLE_SECURITY_PLUGIN: "true"  # 禁用安全插件（开发环境）
    bootstrap.memory_lock: "true"   # 锁定内存以提高性能
  ports:
    - "9200:9200"  # HTTP API
    - "9600:9600"  # 性能监控
  ulimits:
    memlock:
      soft: -1
      hard: -1
  volumes:
    - opensearch_data:/usr/share/opensearch/data
  healthcheck:
    test: ["CMD-SHELL", "curl -f http://localhost:9200/_cluster/health || exit 1"]
    interval: 30s
    timeout: 10s
    retries: 5
    start_period: 60s
```

#### OpenSearch Dashboards

```yaml
opensearch-dashboards:
  image: opensearchproject/opensearch-dashboards:2.19.0
  container_name: rag-dashboards
  ports:
    - "5601:5601"
  environment:
    OPENSEARCH_HOSTS: http://opensearch:9200
    DISABLE_SECURITY_DASHBOARDS_PLUGIN: "true"
  depends_on:
    - opensearch
```

**访问地址**: http://localhost:5601

---

### 5. 🌬️ Apache Airflow

#### 作用
- **工作流编排**: 管理定时任务和依赖关系
- **ETL 流程**: 数据抽取、转换、加载
- **调度**: 定期执行数据摄取任务

#### 核心概念

##### 1. DAG (Directed Acyclic Graph)
有向无环图，定义工作流的依赖关系。

```python
from airflow import DAG
from datetime import datetime, timedelta
from airflow.operators.python import PythonOperator

# 定义 DAG
default_args = {
    'owner': 'rag-system',
    'depends_on_past': False,
    'start_date': datetime(2025, 1, 1),
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 3,
    'retry_delay': timedelta(minutes=5),
}

dag = DAG(
    'arxiv_paper_ingestion',
    default_args=default_args,
    description='Fetch and parse arXiv papers daily',
    schedule_interval='@daily',  # 每天执行
    catchup=False,
)

# 定义任务
fetch_metadata = PythonOperator(
    task_id='fetch_arxiv_metadata',
    python_callable=fetch_papers_from_arxiv,
    dag=dag,
)

download_pdfs = PythonOperator(
    task_id='download_pdf_files',
    python_callable=download_pdfs,
    dag=dag,
)

parse_pdfs = PythonOperator(
    task_id='parse_pdf_content',
    python_callable=parse_pdfs_with_docling,
    dag=dag,
)

# 定义依赖关系
fetch_metadata >> download_pdfs >> parse_pdfs
```

##### 2. 任务依赖
使用 `>>` 操作符定义任务顺序。

```
┌──────────────┐
│  Fetch Meta  │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Download PDFs│
└──────┬───────┘
       │
       ▼
┌──────────────┐
│   Parse PDFs │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│  Index to DB │
└──────────────┘
```

#### 配置示例

```yaml
airflow:
  build:
    context: ./airflow
    dockerfile: Dockerfile
  container_name: rag-airflow
  depends_on:
    postgres:
      condition: service_healthy
  env_file:
    - .env
  environment:
    AIRFLOW_HOME: /opt/airflow
    PYTHONPATH: /opt/airflow/src
    POSTGRES_DATABASE_URL: postgresql+psycopg2://rag_user:rag_password@postgres:5432/rag_db
    OPENSEARCH_HOST: http://opensearch:9200
    OLLAMA_HOST: http://ollama:11434
    REDIS__HOST: redis
  volumes:
    - ./airflow/dags:/opt/airflow/dags      # DAG 文件
    - airflow_logs:/opt/airflow/logs       # 日志
    - ./airflow/plugins:/opt/airflow/plugins
    - ./src:/opt/airflow/src              # 源代码挂载
  ports:
    - "8080:8080"
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
    interval: 30s
    timeout: 10s
    retries: 3
    start_period: 120s
```

**访问地址**: http://localhost:8080

**默认凭据**: 查看 `airflow/simple_auth_manager_passwords.json.generated`

---

### 6. 🦙 Ollama

#### 作用
- **本地 LLM**: 在本地运行大语言模型
- **隐私保护**: 数据不离开本地环境
- **成本节约**: 无需付费 API

#### 支持的模型

```bash
# 拉取模型
ollama pull llama3.2:1b      # 轻量级模型 (~1.3GB)
ollama pull llama3.2:3b      # 中等模型 (~3GB)
ollama pull llama3.2:7b      # 标准模型 (~4.7GB)
ollama pull mistral:7b        # Mistral 模型
ollama pull codellama:7b      # 代码专用模型
```

#### API 使用

```python
import requests

def query_ollama(prompt: str, model: str = "llama3.2:1b"):
    """查询 Ollama 本地模型"""
    response = requests.post(
        "http://localhost:11434/api/generate",
        json={
            "model": model,
            "prompt": prompt,
            "stream": False
        },
        timeout=300
    )
    return response.json()["response"]

# 使用示例
answer = query_ollama(
    prompt="Explain attention mechanism in transformers.",
    model="llama3.2:1b"
)
```

#### 配置示例

```yaml
ollama:
  image: ollama/ollama:0.11.2
  container_name: rag-ollama
  ports:
    - "11434:11434"
  volumes:
    - ollama_data:/root/.ollama  # 模型存储
  healthcheck:
    test: ["CMD", "ollama", "list"]
    interval: 30s
    timeout: 10s
    retries: 3
    start_period: 30s
```

---

### 7. 🚀 Redis

#### 作用
- **缓存**: 缓存频繁访问的数据
- **会话存储**: 存储用户会话信息
- **速率限制**: 控制 API 调用频率
- **发布/订阅**: 消息队列和事件通知

#### 数据结构

```python
import redis

# 连接 Redis
r = redis.Redis(
    host='localhost',
    port=6379,
    password='your_password',
    db=0,
    decode_responses=True
)

# String 类型
r.set('key', 'value', ex=3600)  # 设置过期时间（秒）
value = r.get('key')

# Hash 类型
r.hset('user:1', mapping={'name': 'Alice', 'age': '30'})
name = r.hget('user:1', 'name')

# List 类型
r.lpush('queue', 'task1', 'task2')
task = r.rpop('queue')

# Set 类型
r.sadd('tags', 'ai', 'ml', 'nlp')
tags = r.smembers('tags')
```

#### 缓存模式

```python
from functools import wraps
import hashlib
import json

def cache_result(ttl: int = 3600):
    """缓存装饰器"""
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            # 生成缓存键
            cache_key = f"{func.__name__}:{hash(str(args) + str(kwargs))}"
            
            # 尝试从缓存获取
            cached = r.get(cache_key)
            if cached:
                return json.loads(cached)
            
            # 执行函数
            result = await func(*args, **kwargs)
            
            # 存入缓存
            r.setex(cache_key, ttl, json.dumps(result))
            
            return result
        return wrapper
    return decorator

# 使用示例
@cache_result(ttl=3600)
async def get_paper_content(paper_id: str):
    """获取论文内容（带缓存）"""
    return fetch_from_db(paper_id)
```

#### 配置示例

```yaml
redis:
  image: redis:7-alpine
  container_name: rag-redis
  ports:
    - "6379:6379"
  volumes:
    - redis_data:/data
  command: redis-server --appendonly yes --maxmemory 256mb --maxmemory-policy allkeys-lru
  healthcheck:
    test: ["CMD", "redis-cli", "ping"]
    interval: 10s
    timeout: 3s
    retries: 3
    start_period: 10s
  restart: unless-stopped
```

#### 参数说明

- `--appendonly yes`: 启用 AOF 持久化
- `--maxmemory 256mb`: 最大内存使用
- `--maxmemory-policy allkeys-lru`: 淘汰策略（LRU）

---

### 8. 📊 Langfuse

#### 作用
- **LLM 可观测性**: 追踪 LLM 调用和性能
- **Prompt 管理**: 版本控制和 A/B 测试
- **成本分析**: 监控 API 成本
- **质量评估**: 评估回答质量

#### 核心功能

```python
from langfuse import Langfuse

# 初始化客户端
langfuse = Langfuse(
    public_key="pk-lf-xxx",
    secret_key="sk-lf-xxx",
    host="http://localhost:3001"
)

# 创建追踪
trace = langfuse.trace(
    name="rag_query",
    user_id="user123",
    metadata={"query": "What is attention?"}
)

# 创建生成记录
generation = trace.generation(
    name="llm_response",
    model="llama3.2:1b",
    input="What is attention mechanism?",
    output="Attention is a mechanism that...",
    usage={
        "prompt_tokens": 100,
        "completion_tokens": 200,
        "total_tokens": 300
    },
    metadata={
        "retrieval_time": 0.5,
        "documents_retrieved": 5
    }
)
```

#### 配置示例

```yaml
# ClickHouse (用于存储指标)
clickhouse:
  image: clickhouse/clickhouse-server:24.8-alpine
  container_name: rag-clickhouse
  environment:
    CLICKHOUSE_DB: langfuse
    CLICKHOUSE_USER: langfuse
    CLICKHOUSE_PASSWORD: langfuse
  volumes:
    - clickhouse_data:/var/lib/clickhouse
  healthcheck:
    test: ["CMD", "clickhouse-client", "--query", "SELECT 1"]
    interval: 30s
    timeout: 10s
    retries: 5

# Langfuse Worker
langfuse-worker:
  image: docker.io/langfuse/langfuse-worker:3
  container_name: rag-langfuse-worker
  depends_on:
    langfuse-postgres:
      condition: service_healthy
    langfuse-minio:
      condition: service_healthy
    langfuse-redis:
      condition: service_healthy
    clickhouse:
      condition: service_healthy
  environment:
    DATABASE_URL: postgresql://langfuse:langfuse@langfuse-postgres:5432/langfuse
    SALT: ${LANGFUSE_SALT}
    ENCRYPTION_KEY: ${LANGFUSE_ENCRYPTION_KEY}
    TELEMETRY_ENABLED: "false"
    CLICKHOUSE_URL: http://clickhouse:8123
    REDIS_HOST: langfuse-redis

# Langfuse Web UI
langfuse-web:
  image: docker.io/langfuse/langfuse:3
  container_name: rag-langfuse-web
  ports:
    - "3001:3000"
  environment:
    DATABASE_URL: postgresql://langfuse:langfuse@langfuse-postgres:5432/langfuse
    NEXTAUTH_SECRET: ${LANGFUSE_NEXTAUTH_SECRET}
    SALT: ${LANGFUSE_SALT}
    ENCRYPTION_KEY: ${LANGFUSE_ENCRYPTION_KEY}
    LANGFUSE_INIT_ORG_NAME: "RAG Organization"
    LANGFUSE_INIT_PROJECT_NAME: "Agentic RAG"
    LANGFUSE_INIT_USER_EMAIL: "admin@example.com"
    LANGFUSE_INIT_USER_PASSWORD: "admin123"
```

**访问地址**: http://localhost:3001

**默认凭据**: 
- Email: `admin@example.com`
- Password: `admin123`

---

## Docker Compose 架构

### 🏗️ 完整架构图

```
┌──────────────────────────────────────────────────────────────┐
│                    Docker Network: rag-network               │
└──────────────────────────────────────────────────────────────┘
                          │
      ┌───────────────────┼───────────────────┐
      │                   │                   │
  ┌───▼────┐       ┌────▼────┐       ┌────▼─────┐
  │   API   │       │  Redis  │       │ Postgres │
  │  :8000  │──────▶│  :6379  │◀──────│  :5432   │
  └────┬────┘       └─────────┘       └────┬─────┘
       │                                  │
       │       ┌───────────────────┐        │
       └──────▶│                 │◀───────┘
               │   OpenSearch    │
               │  :9200, :9600  │
               └────────┬────────┘
                        │
        ┌───────────────┼───────────────┐
        │               │               │
   ┌────▼────┐   ┌────▼────┐   ┌────▼────┐
   │Airflow  │   │ Ollama  │   │Langfuse │
   │ :8080   │   │ :11434  │   │ :3001   │
   └─────────┘   └─────────┘   └─────────┘
                                 │
                    ┌────────────┴────────────┐
                    │                         │
               ┌────▼────┐             ┌────▼────┐
               │MinIO    │             │ClickHouse│
               │ :9090   │             │ :8123   │
               └─────────┘             └─────────┘
```

### 📋 服务清单

| 服务 | 端口 | 作用 | 依赖 |
|------|------|------|------|
| api | 8000 | FastAPI API 服务 | postgres, opensearch, redis |
| postgres | 5432 | 主数据库 | - |
| opensearch | 9200, 9600 | 搜索引擎 | - |
| opensearch-dashboards | 5601 | 可视化界面 | opensearch |
| redis | 6379 | 缓存 | - |
| airflow | 8080 | 工作流编排 | postgres |
| ollama | 11434 | 本地 LLM | - |
| langfuse-web | 3001 | 可观测性面板 | clickhouse, minio |
| langfuse-worker | 3030 | 后台任务 | langfuse-web |
| clickhouse | 8123 | 时序数据库 | - |
| langfuse-postgres | 5433 | Langfuse 数据库 | - |
| langfuse-redis | 6380 | Langfuse 缓存 | - |
| langfuse-minio | 9090, 9091 | 对象存储 | - |

### 🔗 依赖关系

```yaml
# 服务依赖图
api:
  depends_on:
    postgres:
      condition: service_healthy
    opensearch:
      condition: service_healthy
    redis:
      condition: service_healthy

opensearch-dashboards:
  depends_on:
    - opensearch

airflow:
  depends_on:
    postgres:
      condition: service_healthy

langfuse-worker:
  depends_on:
    langfuse-postgres:
      condition: service_healthy
    langfuse-minio:
      condition: service_healthy
    langfuse-redis:
      condition: service_healthy
    clickhouse:
      condition: service_healthy
```

### 💾 数据卷 (Volumes)

```yaml
volumes:
  postgres_data:           # PostgreSQL 数据
  opensearch_data:         # OpenSearch 索引
  ollama_data:             # Ollama 模型
  airflow_logs:            # Airflow 日志
  redis_data:              # Redis 数据
  clickhouse_data:         # ClickHouse 数据
  langfuse_v3_postgres_data:  # Langfuse PostgreSQL
  langfuse_v3_minio_data:     # Langfuse 对象存储
```

### 🌐 网络

```yaml
networks:
  rag-network:
    driver: bridge
```

**说明**: 所有服务都在同一个桥接网络中，可以通过服务名称互相访问。

**示例**:
- API 访问数据库: `postgresql://rag_user:rag_password@postgres:5432/rag_db`
- API 访问 OpenSearch: `http://opensearch:9200`
- API 访问 Redis: `redis://redis:6379`

---

## 实践指南

### 📋 前置要求

| 工具 | 版本要求 | 用途 |
|------|---------|------|
| Docker Desktop | 最新版本 | 容器编排 |
| Python | 3.12+ | 开发环境 |
| UV | 最新版本 | 包管理器 |
| 硬件 | 8GB RAM, 20GB 磁盘 | 运行环境 |

### 🚀 安装步骤

#### 1. 安装 UV 包管理器

```bash
# macOS/Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"

# 验证安装
uv --version
```

#### 2. 克隆仓库

```bash
# 克隆主仓库
git clone https://github.com/jamwithai/production-agentic-rag-course.git
cd production-agentic-rag-course

# 或者克隆特定周次（例如 Week 1）
git clone --branch week1.0 https://github.com/jamwithai/production-agentic-rag-course.git
cd production-agentic-rag-course
```

#### 3. 配置环境变量

```bash
# 复制环境变量模板
cp .env.example .env

# 编辑 .env 文件，添加必需的 API 密钥
nano .env
```

**必需的配置**:

```bash
# Jina AI Embeddings API（用于向量搜索）
JINA_API_KEY=your_jina_api_key_here

# Langfuse（用于可观测性）
LANGFUSE_PUBLIC_KEY=pk-lf-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
LANGFUSE_SECRET_KEY=sk-lf-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
LANGFUSE_NEXTAUTH_SECRET=changeme-v3-nextauth-secret-min-32-chars-recommended
LANGFUSE_SALT=changeme-v3-salt-min-32-chars-recommended-for-security
LANGFUSE_ENCRYPTION_KEY=changeme-v3-encryption-key-64-hex-chars-required-see-docs

# Telegram Bot（Week 7）
TELEGRAM__BOT_TOKEN=your_telegram_bot_token_here
```

#### 4. 安装依赖

```bash
# 使用 UV 安装 Python 依赖
uv sync
```

#### 5. 启动所有服务

```bash
# 构建并启动所有容器
docker compose up --build -d

# 查看日志
docker compose logs -f

# 查看特定服务日志
docker compose logs -f api
docker compose logs -f airflow
```

#### 6. 验证服务状态

```bash
# 检查所有容器状态
docker compose ps

# 测试 API 健康检查
curl http://localhost:8000/api/v1/health

# 测试 OpenSearch
curl http://localhost:9200/_cluster/health

# 测试 PostgreSQL
docker exec rag-postgres psql -U rag_user -d rag_db -c "SELECT version();"
```

### 📊 访问各服务

| 服务 | URL | 用途 | 默认凭据 |
|------|-----|------|---------|
| FastAPI Docs | http://localhost:8000/docs | API 文档和测试 | - |
| Airflow Dashboard | http://localhost:8080 | 工作流管理 | 查看密码文件 |
| OpenSearch Dashboards | http://localhost:5601 | 搜索可视化 | - |
| Langfuse Dashboard | http://localhost:3001 | LLM 追踪 | admin@example.com / admin123 |
| Gradio Interface | http://localhost:7861 | 聊天界面 | - |

### 🎓 使用 Jupyter Notebook 学习

```bash
# 启动 Jupyter Notebook
uv run jupyter notebook notebooks/week1/week1_setup.ipynb
```

**Notebook 功能**:

1. ✅ 环境验证
2. ✅ 服务健康监控
3. ✅ 连接测试
4. ✅ 交互式探索
5. ✅ AI 模型设置
6. ✅ 故障排除指南
7. ✅ 生产级最佳实践

### 🛠️ 常用命令

```bash
# Docker Compose 命令
docker compose up -d              # 启动服务（后台）
docker compose down               # 停止服务
docker compose down -v            # 停止服务并删除数据
docker compose restart            # 重启服务
docker compose logs -f           # 查看日志（实时）
docker compose ps                # 查看容器状态
docker compose exec api bash      # 进入容器
docker compose build --no-cache  # 重新构建（无缓存）

# Git 命令
git fetch --all                  # 获取所有分支
git checkout week1.0             # 切换到 Week 1
git pull origin week1.0           # 拉取最新代码

# 数据库命令
docker exec rag-postgres psql -U rag_user -d rag_db  # 进入数据库
docker exec rag-postgres pg_dump -U rag_user rag_db > backup.sql  # 备份
cat backup.sql | docker exec -i rag-postgres psql -U rag_user rag_db  # 恢复

# OpenSearch 命令
curl -X GET http://localhost:9200/_cat/indices?v  # 查看索引
curl -X DELETE http://localhost:9200/arxiv-papers  # 删除索引
```

---

## 配置详解

### 🔧 环境变量配置

#### 应用设置

```bash
DEBUG=true                    # 调试模式（生产环境设为 false）
ENVIRONMENT=development       # 环境（development/staging/production）
```

#### 数据库配置

```bash
# PostgreSQL 连接字符串
POSTGRES_DATABASE_URL=postgresql+psycopg2://rag_user:rag_password@postgres:5432/rag_db

# 参数说明：
# - rag_user: 数据库用户名
# - rag_password: 数据库密码
# - postgres: 主机名（Docker 容器名称）
# - 5432: 端口
# - rag_db: 数据库名
```

#### OpenSearch 配置

```bash
# OpenSearch 主机
OPENSEARCH_HOST=http://opensearch:9200

# 索引配置
OPENSEARCH__INDEX_NAME=arxiv-papers           # 主索引名
OPENSEARCH__CHUNK_INDEX_SUFFIX=chunks         # 分块索引后缀
OPENSEARCH__MAX_TEXT_SIZE=1000000            # 最大文本大小（1MB）

# 向量搜索配置
OPENSEARCH__VECTOR_DIMENSION=1024             # 向量维度（Jina 嵌入）
OPENSEARCH__VECTOR_SPACE_TYPE=cosinesimil     # 相似度类型

# 混合搜索配置
OPENSEARCH__RRF_PIPELINE_NAME=hybrid-rrf-pipeline  # RRF 管道名
OPENSEARCH__HYBRID_SEARCH_SIZE_MULTIPLIER=2         # 搜索结果倍数
```

#### 文本分块配置

```bash
# 分块策略
CHUNKING__CHUNK_SIZE=600              # 每块字符数
CHUNKING__OVERLAP_SIZE=100            # 重叠字符数
CHUNKING__MIN_CHUNK_SIZE=100          # 最小块大小
CHUNKING__SECTION_BASED=true         # 基于章节分块
```

#### arXiv API 配置

```bash
# API 设置
ARXIV__MAX_RESULTS=15                # 每次查询最大结果数
ARXIV__BASE_URL=https://export.arxiv.org/api/query  # API 端点
ARXIV__PDF_CACHE_DIR=./data/arxiv_pdfs  # PDF 缓存目录

# 速率限制
ARXIV__RATE_LIMIT_DELAY=3.0          # 请求间隔（秒）
ARXIV__TIMEOUT_SECONDS=30            # 超时时间

# 重试策略
ARXIV__DOWNLOAD_MAX_RETRIES=3        # 最大重试次数
ARXIV__DOWNLOAD_RETRY_DELAY_BASE=5.0  # 重试基础延迟（秒）

# 并发控制
ARXIV__MAX_CONCURRENT_DOWNLOADS=5    # 最大并发下载数
ARXIV__MAX_CONCURRENT_PARSING=1      # 最大并发解析数

# 搜索设置
ARXIV__SEARCH_CATEGORY=cs.AI         # 搜索类别（计算机科学 - AI）
```

#### PDF 解析配置

```bash
PDF_PARSER__MAX_PAGES=30             # 最大解析页数
PDF_PARSER__MAX_FILE_SIZE_MB=20      # 最大文件大小（MB）
PDF_PARSER__DO_OCR=false             # 是否启用 OCR
PDF_PARSER__DO_TABLE_STRUCTURE=true  # 是否提取表格结构
```

#### Ollama 配置

```bash
OLLAMA_HOST=http://ollama:11434      # Ollama 服务地址
OLLAMA_MODEL=llama3.2:1b            # 默认模型
OLLAMA_TIMEOUT=300                   # 超时时间（秒）
```

#### Redis 配置

```bash
REDIS__HOST=redis                    # Redis 主机
REDIS__PORT=6379                    # Redis 端口
REDIS__PASSWORD=your_password        # Redis 密码
REDIS__DB=0                         # Redis 数据库编号
REDIS__TTL_HOURS=6                  # 缓存过期时间（小时）
```

#### Langfuse 配置

```bash
# Langfuse 客户端设置
LANGFUSE_ENABLED=true                # 是否启用追踪
LANGFUSE_HOST=http://localhost:3001  # Langfuse 服务地址
LANGFUSE_PUBLIC_KEY=pk-lf-xxx       # 公钥
LANGFUSE_SECRET_KEY=sk-lf-xxx       # 私钥
LANGFUSE_FLUSH_AT=15                 # 每 15 次调用刷新一次
LANGFUSE_FLUSH_INTERVAL=1.0          # 每 1 秒刷新一次
LANGFUSE_DEBUG=true                  # 调试模式

# Langfuse 服务器配置（用于自部署）
LANGFUSE_NEXTAUTH_SECRET=changeme-xxx
LANGFUSE_SALT=changeme-xxx
LANGFUSE_ENCRYPTION_KEY=changeme-xxx
LANGFUSE_REDIS_PASSWORD=langfuse_redis_password
LANGFUSE_MINIO_ACCESS_KEY=langfuse_minio
LANGFUSE_MINIO_SECRET_KEY=langfuse_minio_secret
```

#### Airflow 配置

```bash
# 核心设置
AIRFLOW__CORE__EXECUTOR=LocalExecutor           # 执行器类型
AIRFLOW__CORE__LOAD_EXAMPLES=false            # 不加载示例 DAG
AIRFLOW__WEBSERVER__EXPOSE_CONFIG=true        # 暴露配置（开发环境）

# 数据库连接
AIRFLOW__DATABASE__SQL_ALCHEMY_CONN=postgresql+psycopg2://rag_user:rag_password@postgres:5432/rag_db

# 工作目录
AIRFLOW_HOME=/opt/airflow
```

---

## 常见问题与解决方案

### ❓ 问题 1: Docker Compose 启动失败

**症状**:
```bash
$ docker compose up -d
ERROR: The Compose file './compose.yml' is invalid
```

**解决方案**:

1. 检查 Docker 版本
```bash
docker --version  # 需要 20.10+
docker compose version
```

2. 检查端口占用
```bash
# macOS/Linux
lsof -i :8000
lsof -i :5432

# 关闭占用端口的服务
kill -9 <PID>
```

3. 清理旧容器
```bash
docker compose down -v
docker system prune -a
```

### ❓ 问题 2: PostgreSQL 连接失败

**症状**:
```bash
ConnectionError: could not connect to server: Connection refused
```

**解决方案**:

1. 检查容器状态
```bash
docker compose ps postgres
docker compose logs postgres
```

2. 检查连接字符串
```bash
# 确保使用容器名称作为主机名
POSTGRES_DATABASE_URL=postgresql+psycopg2://rag_user:rag_password@postgres:5432/rag_db
```

3. 测试连接
```bash
docker exec rag-postgres psql -U rag_user -d rag_db -c "SELECT 1;"
```

### ❓ 问题 3: OpenSearch 内存不足

**症状**:
```bash
OpenSearchException: OpenSearchException[Unable to start: max virtual memory areas vm.max_map_count [65530] is too low
```

**解决方案**:

1. **macOS/Docker Desktop**: 在 Docker 设置中增加内存分配（推荐 4GB+）

2. **Linux**: 调整内核参数
```bash
sudo sysctl -w vm.max_map_count=262144
# 永久生效
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
```

3. 调整 OpenSearch JVM 内存
```yaml
environment:
  OPENSEARCH_JAVA_OPTS: "-Xms512m -Xmx1024m"  # 增加堆内存
```

### ❓ 问题 4: Airflow WebUI 无法访问

**症状**:
```bash
$ curl http://localhost:8080/health
curl: (7) Failed to connect to localhost port 8080: Connection refused
```

**解决方案**:

1. 等待 Airflow 初始化（可能需要 2-3 分钟）
```bash
docker compose logs -f airflow
# 看到 "Running on http://0.0.0.0:8080" 说明启动成功
```

2. 检查 Airflow 密码
```bash
cat airflow/simple_auth_manager_passwords.json.generated
# 记录 admin 用户密码
```

3. 重启 Airflow
```bash
docker compose restart airflow
```

### ❓ 问题 5: Ollama 模型下载失败

**症状**:
```bash
Error: failed to pull model: connection refused
```

**解决方案**:

1. 手动下载模型
```bash
docker exec -it rag-ollama ollama pull llama3.2:1b
```

2. 检查磁盘空间
```bash
df -h  # 确保有足够空间
```

3. 使用国内镜像（如果遇到网络问题）
```bash
# 设置 OLLAMA_HOST 环境变量指向镜像
export OLLAMA_HOST=http://mirror.ollama.com:11434
```

### ❓ 问题 6: Langfuse 数据库迁移失败

**症状**:
```bash
DatabaseError: relation "users" does not exist
```

**解决方案**:

1. 重启 Langfuse 服务（自动运行迁移）
```bash
docker compose restart langfuse-web
docker compose restart langfuse-worker
```

2. 手动运行迁移
```bash
docker exec -it rag-langfuse-worker alembic upgrade head
```

3. 检查数据库连接
```bash
docker exec -it rag-langfuse-postgres psql -U langfuse -d langfuse -c "\dt"
```

### ❓ 问题 7: 容器日志过多占用磁盘空间

**解决方案**:

1. 查看日志大小
```bash
docker system df
```

2. 清理日志
```bash
# 方案 1: 清理所有日志
docker system prune -a

# 方案 2: 只清理未使用的日志
docker system prune

# 方案 3: 限制日志大小（在 compose.yml 中添加）
services:
  api:
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

### ❓ 问题 8: 网络连接问题

**症状**: 容器之间无法互相访问

**解决方案**:

1. 检查网络
```bash
docker network ls
docker network inspect rag_rag-network
```

2. 重新创建网络
```bash
docker compose down
docker network prune
docker compose up -d
```

3. 使用容器名称而非 IP
```bash
# ✅ 正确
OPENSEARCH_HOST=http://opensearch:9200

# ❌ 错误
OPENSEARCH_HOST=http://172.18.0.5:9200
```

---

## 学习检查清单

### ✅ Week 1 完成标准

#### 环境准备
- [ ] Docker Desktop 已安装并运行
- [ ] Python 3.12+ 已安装
- [ ] UV 包管理器已安装
- [ ] Git 已克隆仓库

#### 服务启动
- [ ] 所有 13 个容器正在运行（`docker compose ps`）
- [ ] API 服务响应正常（`curl http://localhost:8000/api/v1/health`）
- [ ] PostgreSQL 可以连接并查询
- [ ] OpenSearch 集群健康
- [ ] Airflow Dashboard 可访问
- [ ] Ollama 已下载模型
- [ ] Redis 可以存储和检索数据
- [ ] Langfuse Dashboard 可访问

#### 功能验证
- [ ] API 文档可访问（http://localhost:8000/docs）
- [ ] OpenSearch Dashboards 可访问
- [ ] 可以通过 Airflow 创建和运行 DAG
- [ ] Ollama 可以生成文本
- [ ] Redis 缓存正常工作
- [ ] Langfuse 可以追踪 API 调用

#### 理解检查
- [ ] 理解 Docker Compose 的作用
- [ ] 理解 FastAPI 的核心概念
- [ ] 理解 PostgreSQL 的连接池管理
- [ ] 理解 OpenSearch 的 BM25 和向量搜索
- [ ] 理解 Airflow 的 DAG 和任务依赖
- [ ] 理解 Redis 的缓存策略
- [ ] 理解 Langfuse 的可观测性功能

#### 实践任务
- [ ] 完成 week1_setup.ipynb 笔记本
- [ ] 测试所有服务的健康检查
- [ ] 创建一个简单的 API 端点
- [ ] 在 PostgreSQL 中创建一个表
- [ ] 在 OpenSearch 中创建一个索引
- [ ] 在 Airflow 中创建一个简单的 DAG
- [ ] 使用 Ollama 生成文本
- [ ] 使用 Redis 缓存数据
- [ ] 在 Langfuse 中查看追踪数据

### 🎯 进阶挑战

1. **性能优化**: 调整 OpenSearch 的 JVM 内存参数，对比查询性能
2. **监控配置**: 配置 Prometheus + Grafana 监控所有服务
3. **备份恢复**: 实现数据库和索引的自动备份
4. **负载测试**: 使用 k6 或 locust 对 API 进行压力测试
5. **日志聚合**: 配置 ELK Stack 或 Loki 收集和分析日志

---

## 深度实践与进阶内容

### 🔧 实战代码示例

#### 1. FastAPI 完整 CRUD 示例

```python
# models.py - 数据模型
from sqlalchemy import Column, Integer, String, DateTime
from sqlalchemy.ext.declarative import declarative_base
from datetime import datetime

Base = declarative_base()

class Paper(Base):
    __tablename__ = "papers"
    
    id = Column(Integer, primary_key=True, index=True)
    arxiv_id = Column(String(50), unique=True, nullable=False, index=True)
    title = Column(String, nullable=False)
    abstract = Column(String)
    authors = Column(String)  # JSONB
    published_date = Column(DateTime)
    created_at = Column(DateTime, default=datetime.utcnow)

# database.py - 数据库连接
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

DATABASE_URL = "postgresql+psycopg2://rag_user:rag_password@postgres:5432/rag_db"

engine = create_engine(
    DATABASE_URL,
    pool_size=10,
    max_overflow=20,
    pool_timeout=30,
    pool_recycle=3600,
    pool_pre_ping=True
)

SessionLocal = sessionmaker(bind=engine)

# main.py - FastAPI 应用
from fastapi import FastAPI, Depends, HTTPException, Query
from sqlalchemy.orm import Session
from typing import List, Optional
import uvicorn

app = FastAPI(
    title="ArXiv Paper API",
    description="Production-grade RAG system API",
    version="1.0.0"
)

# 依赖注入
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

# CRUD 端点
@app.post("/api/v1/papers")
async def create_paper(paper: PaperSchema, db: Session = Depends(get_db)):
    """创建新论文"""
    db_paper = Paper(**paper.dict())
    db.add(db_paper)
    db.commit()
    db.refresh(db_paper)
    return db_paper

@app.get("/api/v1/papers")
async def list_papers(
    skip: int = 0,
    limit: int = 10,
    search: Optional[str] = None,
    db: Session = Depends(get_db)
):
    """获取论文列表（支持搜索和分页）"""
    query = db.query(Paper)
    
    if search:
        query = query.filter(Paper.title.ilike(f"%{search}%"))
    
    papers = query.offset(skip).limit(limit).all()
    total = query.count()
    
    return {
        "items": papers,
        "total": total,
        "skip": skip,
        "limit": limit
    }

@app.get("/api/v1/papers/{paper_id}")
async def get_paper(paper_id: int, db: Session = Depends(get_db)):
    """获取单个论文"""
    paper = db.query(Paper).filter(Paper.id == paper_id).first()
    if not paper:
        raise HTTPException(status_code=404, detail="Paper not found")
    return paper

# 启动服务器
if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

#### 2. OpenSearch 混合搜索示例

```python
from opensearchpy import OpenSearch, helpers

class OpenSearchClient:
    def __init__(self, host="http://opensearch:9200"):
        self.client = OpenSearch(hosts=[host])
    
    def create_index(self, index_name):
        """创建索引"""
        if not self.client.indices.exists(index=index_name):
            self.client.indices.create(
                index=index_name,
                body={
                    "settings": {
                        "number_of_shards": 1,
                        "number_of_replicas": 0
                    },
                    "mappings": {
                        "properties": {
                            "title": {"type": "text"},
                            "abstract": {"type": "text"},
                            "content": {"type": "text"},
                            "authors": {"type": "keyword"},
                            "vector_embedding": {
                                "type": "knn_vector",
                                "dimension": 1024,
                                "method": {
                                    "name": "hnsw",
                                    "space_type": "cosinesimil",
                                    "engine": "nmslib"
                                }
                            }
                        }
                    }
                }
            )
    
    def hybrid_search(self, query, vector, index_name, k=10):
        """混合搜索（BM25 + 向量）"""
        search_body = {
            "size": k,
            "query": {
                "bool": {
                    "should": [
                        # BM25 关键词搜索
                        {
                            "multi_match": {
                                "query": query,
                                "fields": ["title^2", "abstract", "content"],
                                "type": "best_fields"
                            }
                        },
                        # 向量搜索
                        {
                            "knn": {
                                "vector_embedding": {
                                    "vector": vector,
                                    "k": k
                                }
                            }
                        }
                    ]
                }
            }
        }
        
        response = self.client.search(index=index_name, body=search_body)
        return response['hits']['hits']

# 使用示例
es_client = OpenSearchClient()
es_client.create_index("papers")

# 执行混合搜索
results = es_client.hybrid_search(
    query="attention mechanism",
    vector=[0.1, 0.2, 0.3, ...],  # 1024 维向量
    index_name="papers",
    k=10
)
```

#### 3. Redis 缓存装饰器（完整版）

```python
import redis
import json
import hashlib
import functools
from typing import Optional, Any

class RedisCache:
    def __init__(self, host='redis', port=6379, db=0):
        self.client = redis.Redis(
            host=host,
            port=port,
            db=db,
            decode_responses=True
        )
    
    def _generate_key(self, func_name: str, args: tuple, kwargs: dict) -> str:
        """生成缓存键"""
        args_str = str(args) + str(sorted(kwargs.items()))
        key_hash = hashlib.md5(args_str.encode()).hexdigest()
        return f"{func_name}:{key_hash}"
    
    def cache(self, ttl: int = 3600, key_prefix: Optional[str] = None):
        """缓存装饰器"""
        def decorator(func):
            @functools.wraps(func)
            async def wrapper(*args, **kwargs):
                # 生成缓存键
                cache_key = self._generate_key(func.__name__, args, kwargs)
                if key_prefix:
                    cache_key = f"{key_prefix}:{cache_key}"
                
                # 尝试从缓存获取
                cached = self.client.get(cache_key)
                if cached:
                    return json.loads(cached)
                
                # 执行函数
                result = await func(*args, **kwargs)
                
                # 存入缓存
                self.client.setex(
                    cache_key,
                    ttl,
                    json.dumps(result, default=str)
                )
                
                return result
            return wrapper
        return decorator
    
    def invalidate_pattern(self, pattern: str):
        """根据模式清除缓存"""
        keys = self.client.keys(f"*{pattern}*")
        if keys:
            self.client.delete(*keys)

# 使用示例
cache = RedisCache()

@cache.cache(ttl=3600, key_prefix="paper")
async def get_paper(paper_id: str):
    """获取论文（带 1 小时缓存）"""
    # 模拟数据库查询
    return {"id": paper_id, "title": "Attention is All You Need"}

# 清除特定缓存
cache.invalidate_pattern("paper")
```

---

### ⚡ 性能优化技巧

#### 1. PostgreSQL 查询优化

```sql
-- ✅ 正确：使用索引
SELECT * FROM papers 
WHERE arxiv_id = '2301.07041'
AND published_date > '2024-01-01';

-- ❌ 错误：全表扫描
SELECT * FROM papers 
WHERE LOWER(title) LIKE '%attention%';

-- ✅ 优化：添加全文搜索索引
CREATE INDEX idx_papers_title_gin ON papers USING GIN (to_tsvector('english', title));

-- 使用全文搜索
SELECT * FROM papers 
WHERE to_tsvector('english', title) @@ to_tsquery('english', 'attention');
```

#### 2. OpenSearch 性能调优

```yaml
# docker-compose.yml 中的优化配置
opensearch:
  environment:
    # JVM 内存配置
    OPENSEARCH_JAVA_OPTS: "-Xms1g -Xmx1g"
    
    # 索引刷新间隔（提高写入性能）
    index.refresh_interval: "30s"
    
    # 分片配置
    number_of_shards: 1
    number_of_replicas: 0
    
    # 缓存配置
    indices.queries.cache.size: "20%"
    indices.request.cache.size: "2%"
    indices.fielddata.cache.size: "20%"
```

#### 3. FastAPI 并发优化

```python
from fastapi.concurrency import (
    contextmanager_acquire_lock,
    contextmanager_concurrency
)

# 使用并发限制
@app.post("/api/v1/process")
@contextmanager_concurrency(limit=10)  # 最多 10 个并发请求
async def process_data(data: dict):
    """处理数据（带并发限制）"""
    # 模拟耗时操作
    await asyncio.sleep(2)
    return {"status": "completed"}

# 使用锁（防止竞态条件）
@app.post("/api/v1/update")
async def update_record(record_id: int):
    """更新记录（原子操作）"""
    with contextmanager_acquire_lock("update_lock"):
        # 在锁内执行操作
        # 这里确保同一时间只有一个请求能执行
        pass
```

#### 4. Redis 内存优化

```python
# 使用压缩存储大对象
import gzip
import pickle

def set_compressed(key: str, value: Any, ttl: int = 3600):
    """存储压缩对象"""
    serialized = pickle.dumps(value)
    compressed = gzip.compress(serialized)
    r.setex(key, ttl, compressed)

def get_compressed(key: str) -> Optional[Any]:
    """获取并解压对象"""
    compressed = r.get(key)
    if compressed:
        decompressed = gzip.decompress(compressed)
        return pickle.loads(decompressed)
    return None

# 使用示例
large_data = {"text": "..." * 10000}
set_compressed("large_data", large_data, ttl=3600)
```

---

### 🎯 设计模式与反模式

#### ✅ 推荐模式

##### 1. 依赖注入模式
```python
# ✅ 好的做法
from fastapi import Depends

class Database:
    def __init__(self):
        self.connection = create_connection()

def get_db() -> Database:
    return Database()

@app.get("/api/items")
def get_items(db: Database = Depends(get_db)):
    return db.query_items()
```

##### 2. 仓储模式（Repository Pattern）
```python
# ✅ 好的做法：封装数据访问
class PaperRepository:
    def __init__(self, db: Session):
        self.db = db
    
    def get_by_id(self, paper_id: int) -> Optional[Paper]:
        return self.db.query(Paper).filter(Paper.id == paper_id).first()
    
    def search(self, query: str) -> List[Paper]:
        return self.db.query(Paper).filter(Paper.title.ilike(f"%{query}%")).all()

# 使用
@app.get("/api/papers/{paper_id}")
def get_paper(paper_id: int, db: Session = Depends(get_db)):
    repo = PaperRepository(db)
    return repo.get_by_id(paper_id)
```

##### 3. 单元测试友好设计
```python
# ✅ 好的做法：可测试
class SearchService:
    def __init__(self, es_client: OpenSearchClient):
        self.es_client = es_client
    
    def search(self, query: str) -> List[dict]:
        return self.es_client.search(query)

# 测试时可以注入 Mock 对象
def test_search():
    mock_es = Mock()
    service = SearchService(mock_es)
    results = service.search("test")
    assert len(results) > 0
```

#### ❌ 反模式

##### 1. 全局状态
```python
# ❌ 糟糕的做法
db_connection = None

def init_db():
    global db_connection
    db_connection = create_connection()

@app.get("/api/items")
def get_items():
    return db_connection.query_items()

# 问题：难以测试、并发不安全
```

##### 2. 嵌套 API 调用
```python
# ❌ 糟糕的做法
@app.get("/api/user/{user_id}/papers")
def get_user_papers(user_id: int):
    # 第一个 API 调用
    user = call_external_api(f"/users/{user_id}")
    
    # 第二个 API 调用（N+1 问题）
    papers = []
    for paper_id in user['paper_ids']:
        paper = call_external_api(f"/papers/{paper_id}")
        papers.append(paper)
    
    return papers

# ✅ 好的做法：批量查询
@app.get("/api/user/{user_id}/papers")
def get_user_papers(user_id: int):
    user = call_external_api(f"/users/{user_id}?include=papers")
    return user['papers']
```

##### 3. 忽略错误处理
```python
# ❌ 糟糕的做法
@app.post("/api/upload")
def upload_file(file: UploadFile):
    content = file.file.read()
    result = process(content)  # 可能抛出异常
    return result

# ✅ 好的做法：完整的错误处理
@app.post("/api/upload")
def upload_file(file: UploadFile):
    try:
        content = file.file.read()
        result = process(content)
        return result
    except ValueError as e:
        raise HTTPException(status_code=400, detail=str(e))
    except Exception as e:
        logger.error(f"Upload failed: {e}")
        raise HTTPException(status_code=500, detail="Internal server error")
```

---

### 📈 学习路径建议

#### 🌱 初学者路径（2-3 个月）

```
Week 1-2: 基础设施
├─ Docker & Docker Compose 掌握
├─ FastAPI 基础开发
├─ PostgreSQL 基础操作
└─ Redis 缓存使用

Week 3-4: 搜索与数据
├─ OpenSearch 基础
├─ 数据模型设计
└─ 简单的 API 实现

Week 5-8: 进阶功能
├─ OpenSearch 高级（混合搜索）
├─ Airflow 工作流
├─ 性能优化
└─ 监控和日志
```

#### 🚀 进阶路径（1-2 个月）

```
Week 1: 快速搭建
├─ 跳过基础，直接配置生产环境
├─ 完整的 Docker Compose 配置
└─ 基础服务部署

Week 2-3: 核心功能
├─ RAG 管道实现
├─ 混合搜索优化
└─ 缓存策略设计

Week 4-8: 生产化
├─ 监控和告警
├─ 性能调优
├─ 安全加固
└─ CI/CD 流程
```

---

### 🎓 认证与技能评估

#### Week 1 技能检查表

| 技能 | 入门 | 熟练 | 专家 |
|------|------|------|------|
| Docker | 能运行容器 | 编写 Dockerfile | 多阶段构建优化 |
| Docker Compose | 启动服务 | 多服务编排 | 生产网络配置 |
| FastAPI | 基础 API | 依赖注入/中间件 | 异步优化 |
| PostgreSQL | 基础 CRUD | 索引优化 | 查询计划分析 |
| OpenSearch | 索引文档 | 混合搜索 | 性能调优 |
| Redis | 基本缓存 | 缓存策略 | 集群配置 |
| Airflow | 运行 DAG | 动态 DAG | 自定义 Operator |

#### 实战项目建议

**Level 1: 简单项目**
- ✅ 创建一个待办事项 API（FastAPI + PostgreSQL）
- ✅ 实现简单的全文搜索（PostgreSQL 全文搜索）
- ✅ 添加 Redis 缓存层

**Level 2: 中级项目**
- ✅ 构建 RAG 系统（使用 LlamaIndex）
- ✅ 集成 OpenSearch 混合搜索
- ✅ 实现用户认证和权限

**Level 3: 高级项目**
- ✅ 完整的生产级 RAG 系统
- ✅ Airflow 工作流编排
- ✅ Langfuse 可观测性集成
- ✅ K8s 部署

---

## 延伸阅读与推荐资源

### 📖 官方文档与教程

#### Docker & 容器化
- [🐳 Docker 官方文档](https://docs.docker.com/) - 最权威的 Docker 学习资源
- [Docker Compose 指南](https://docs.docker.com/compose/) - 多容器编排完整指南
- [Docker 最佳实践](https://docs.docker.com/develop/dev-best-practices/) - 生产环境部署建议
- [Docker 入门教程](https://docs.docker.com/get-started/) - 从零开始学习 Docker

#### FastAPI
- [⚡ FastAPI 官方文档](https://fastapi.tiangolo.com/) - FastAPI 权威参考
- [FastAPI 用户指南](https://fastapi.tiangolo.com/tutorial/) - 逐步教程，涵盖所有功能
- [FastAPI 高级教程](https://fastapi.tiangolo.com/advanced/) - 依赖注入、后台任务、GraphQL 等
- [Pydantic 文档](https://docs.pydantic.dev/) - FastAPI 的数据验证库

#### PostgreSQL
- [🗄️ PostgreSQL 官方文档](https://www.postgresql.org/docs/) - 数据库完整手册
- [PostgreSQL 教程](https://www.postgresql.org/docs/current/tutorial-start.html) - 入门到精通
- [SQLAlchemy 文档](https://docs.sqlalchemy.org/) - Python SQL 工具包
- [PostgreSQL 性能调优](https://wiki.postgresql.org/wiki/Performance_Optimization) - 性能优化指南

#### OpenSearch & Elasticsearch
- [🔍 OpenSearch 官方文档](https://opensearch.org/docs/latest/) - 搜索引擎完整文档
- [OpenSearch 入门](https://opensearch.org/docs/latest/getting-started/index.html) - 快速开始指南
- [BM25 算法详解](https://en.wikipedia.org/wiki/Okapi_BM25) - 经典排序算法
- [向量搜索原理](https://www.pinecone.io/learn/vector-search/) - 现代搜索技术
- [OpenSearch vs Elasticsearch](https://opensearch.org/what-is-opensearch/) - 两者对比

#### Apache Airflow
- [🌬️ Airflow 官方文档](https://airflow.apache.org/docs/apache-airflow/stable/) - 工作流编排权威指南
- [Airflow 教程](https://airflow.apache.org/docs/apache-airflow/stable/tutorial.html) - 从零开始
- [Airflow 最佳实践](https://airflow.apache.org/docs/apache-airflow/stable/best-practices.html) - 生产环境建议
- [Airflow DAG 编写指南](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/dags.html) - DAG 设计模式

#### Ollama
- [🦙 Ollama 官方网站](https://ollama.com/) - 本地 LLM 运行平台
- [Ollama 模型库](https://ollama.com/library) - 可用模型列表
- [Ollama API 文档](https://github.com/ollama/ollama/blob/main/docs/api.md) - REST API 使用方法
- [Ollama 教程](https://ollama.com/blog/ollama-is-now-available-as-an-open-source-llm-serving-tool) - 深度解析

#### Redis
- [🚀 Redis 官方文档](https://redis.io/docs/) - 内存数据库权威指南
- [Redis 命令参考](https://redis.io/commands/) - 所有 Redis 命令
- [Redis 最佳实践](https://redis.io/topics/lru-cache) - 缓存策略和性能优化
- [Redis 性能优化](https://redis.io/topics/admin) - 生产环境调优

#### Langfuse
- [📊 Langfuse 官方文档](https://langfuse.com/docs) - LLM 可观测性平台
- [Langfuse 快速开始](https://langfuse.com/docs/get-started) - 5 分钟入门
- [LLM 可观测性指南](https://langfuse.com/llm-observability) - 追踪和监控最佳实践
- [Langfuse 示例项目](https://github.com/langfuse/langfuse-python/tree/main/examples) - 实战代码

---

### 🎥 视频教程推荐

#### Docker 系列
- [Docker 入门教程](https://www.youtube.com/watch?v=3c-iBn73dDE) - 15 分钟快速入门
- [Docker Compose 实战](https://www.youtube.com/watch?v=Qw9zP0YH8k) - 多服务编排实战
- [Docker 最佳实践](https://www.youtube.com/watch?v=FlVh1FhO3o) - 生产环境部署技巧

#### FastAPI 系列
- [FastAPI 快速入门](https://www.youtube.com/watch?v=7t2al7EJWJE) - Tiangolo 官方教程
- [FastAPI + PostgreSQL](https://www.youtube.com/watch?v=0sOvQWF_bsY) - 完整 CRUD 应用
- [FastAPI 性能优化](https://www.youtube.com/watch?v=YdA4YgR8t0) - 异步和并发处理

#### 数据库系列
- [PostgreSQL 完整教程](https://www.youtube.com/playlist?list=PLQH6-FVGJlwT-1zR5lR4vGjQ8u5xS) - 从基础到高级
- [SQLAlchemy 教程](https://www.youtube.com/watch?v=wuo-4-IK2qE) - Python ORM 框架
- [Redis 缓存实战](https://www.youtube.com/watch?v=hxwD1i9Y5M0) - Redis 在实际项目中的应用

#### 搜索引擎系列
- [Elasticsearch/OpenSearch 教程](https://www.youtube.com/watch?v=w8pJ4GKQ7k) - 搜索引擎基础
- [向量搜索实战](https://www.youtube.com/watch?v=0yGdCwI5wU) - 语义搜索实现
- [BM25 算法详解](https://www.youtube.com/watch?v=-XQ6zF8DqY) - 排序算法原理

#### 工作流编排系列
- [Airflow 入门教程](https://www.youtube.com/watch?v=BQdVjeJ7-fk) - DAG 和任务编排
- [Airflow 生产部署](https://www.youtube.com/watch?v=cKl3Tl6q-2w) - 大规模数据处理

---

### 📚 优质博客与文章

#### 基础设施与 DevOps
- [The Infrastructure That Powers RAG Systems](https://jamwithai.substack.com/p/the-infrastructure-that-powers-rag) - 本项目的配套博客
- [12 Factor App](https://12factor.net/) - 现代应用设计原则
- [Docker vs VM](https://www.docker.com/why-docker/) - 为什么选择容器化

#### FastAPI 深度解析
- [FastAPI 性能优化](https://fastapi.tiangolo.com/async/) - 异步编程详解
- [FastAPI 安全最佳实践](https://fastapi.tiangolo.com/tutorial/security/) - 认证和授权

#### 数据库优化
- [PostgreSQL 索引优化](https://www.postgresql.org/docs/current/indexes.html) - 索引策略
- [PostgreSQL 连接池管理](https://docs.sqlalchemy.org/en/14/core/pooling.html) - 连接池配置
- [Redis 缓存模式](https://redis.io/topics/patterns/) - 常见缓存模式

#### RAG 与搜索
- [向量搜索原理](https://www.pinecone.io/learn/vector-search/) - 深入理解向量搜索
- [混合搜索策略](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-search.html) - 多搜索融合
- [RAG 最佳实践](https://www.anthropic.com/guides/rag) - 生产级 RAG 设计

---

### 💻 实战项目与案例

#### 完整 RAG 系统
- [🎯 jamwithai/production-agentic-rag-course](https://github.com/jamwithai/production-agentic-rag-course) - 本课程源码
- [LlamaIndex Examples](https://github.com/run-llama/llama_index/tree/main/examples) - 50+ RAG 示例
- [LangChain RAG Tutorial](https://python.langchain.com/docs/tutorials/rag/) - 官方教程

#### 微服务架构
- [Microservices Pattern](https://microservices.io/patterns/) - 微服务设计模式
- [API Gateway 设计](https://nginx.org/en/docs/beginners-guide.html) - 网关架构

#### 监控与可观测性
- [Prometheus + Grafana](https://prometheus.io/docs/visualization/grafana/) - 监控栈
- [ELK Stack](https://www.elastic.co/what-is/elk-stack) - 日志聚合

---

### 🛠️ 工具与资源推荐

#### 开发工具
- [VS Code](https://code.visualstudio.com/) - 推荐的 IDE
- [Docker Desktop](https://www.docker.com/products/docker-desktop) - 本地容器开发
- [TablePlus](https://tableplus.com/) - 数据库管理工具
- [Postman](https://www.postman.com/) - API 测试工具

#### 学习平台
- [Kaggle](https://www.kaggle.com/) - 数据科学竞赛
- [DeepLearning.AI](https://www.deeplearning.ai/) - AI 课程
- [Fast.ai](https://www.fast.ai/) - 深度学习实战课程

#### 社区与论坛
- [Stack Overflow](https://stackoverflow.com/questions/tagged/docker) - 技术问答
- [Reddit r/docker](https://www.reddit.com/r/docker) - Docker 社区讨论
- [GitHub Discussions](https://github.com/jamwithai/production-agentic-rag-course/discussions) - 课程讨论区

### 🎥 视频教程

- [Docker 入门教程](https://www.youtube.com/watch?v=3c-iBn73dDE)
- [FastAPI 快速入门](https://www.youtube.com/watch?v=7t2al7EJWJE)
- [PostgreSQL 完整教程](https://www.youtube.com/watch?v=qw--VYGpxK0)
- [Apache Airflow 教程](https://www.youtube.com/watch?v=BQdVjeJ7-fk)

### 💻 实战项目

1. **构建自己的 RAG 系统**: 基于 Week 1 基础，使用自己的数据集
2. **实时数据管道**: 结合 Kafka 实现实时数据处理
3. **多语言支持**: 添加中文、日文等多语言支持
4. **API 网关**: 使用 Kong 或 Traefik 构建 API 网关
5. **微服务架构**: 将单体应用拆分为微服务

---

## 🎓 学习认证与证书

### 相关认证考试

| 认证 | 难度 | 相关性 | 推荐度 |
|--------|--------|---------|----------|
| Docker Certified Associate | ⭐⭐ | ⭐⭐⭐⭐⭐ | ✅ 强烈推荐 |
| AWS Certified Solutions Architect | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ✅ 推荐 |
| FastAPI（无官方认证）| - | ⭐⭐⭐⭐⭐⭐ | - |
| PostgreSQL CE 12/13/14/15/16 | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ✅ 推荐 |
| Elastic Certified Engineer | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ✅ 推荐 |

### 学习路径建议

**对于想要从事 AI/LLM 开发的开发者：**

```
第一步：容器化和部署 (2-3 周)
├─ Docker 认证
├─ Docker Compose 实战
└─ K8s 基础（可选）

第二步：后端开发 (3-4 周)
├─ FastAPI 深度学习
├─ PostgreSQL 高级
└─ Redis 缓存设计

第三步：RAG 与搜索 (4-6 周)
├─ OpenSearch 实战
├─ 向量数据库
└─ 混合搜索优化

第四步：生产化 (2-3 周)
├─ 监控和告警
├─ CI/CD 流程
└─ 性能优化
```

---

## 🌍 社区与资源

### 中文社区

- [Docker 中文社区](https://www.docker.org.cn/) - Docker 中文资源
- [FastAPI 中文文档](https://fastapi.tiangolo.com/zh/) - FastAPI 中文教程
- [PostgreSQL 中文社区](https://www.postgres.cn/) - PostgreSQL 中文论坛
- [Redis 中文文档](https://redis.io/docs/) - Redis 中文资源

### 国际社区

- [Stack Overflow - Docker](https://stackoverflow.com/questions/tagged/docker)
- [Stack Overflow - FastAPI](https://stackoverflow.com/questions/tagged/fastapi)
- [Stack Overflow - PostgreSQL](https://stackoverflow.com/questions/tagged/postgresql)
- [Reddit r/docker](https://www.reddit.com/r/docker) - Docker 讨论
- [Hacker News](https://news.ycombinator.com/) - 技术新闻

### 开源项目参考

- [realworld/FastAPI](https://github.com/tiangolo/full-stack-fastapi-postgresql) - 完整的全栈示例
- [docker-awesome-compose](https://github.com/veggiemonk/awesome-docker-compose) - Docker Compose 示例集合
- [awesome-production-rag](https://github.com/karpathy/awesome-rag) - RAG 系统集合

---

## 📝 笔记与总结模板

### 学习笔记模板

```markdown
# [日期] - 学习笔记

## 今日目标
- [ ] 理解 Docker Compose
- [ ] 实现第一个 FastAPI 端点
- [ ] 配置 PostgreSQL 连接池

## 学到的概念
1. Docker Compose 的作用
   - 编排多个容器
   - 统一网络管理
   - 数据卷共享

2. FastAPI 的优势
   - 性能卓越
   - 自动文档
   - 类型安全

## 代码示例
```python
# 这里放置重要代码
```

## 遇到的问题
### 问题 1: 容器无法启动
**症状**: ...
**解决**: ...

## 明日计划
- [ ] 深入学习 OpenSearch
- [ ] 实现缓存装饰器
- [ ] 性能测试

## 参考资料
- [链接 1](url)
- [链接 2](url)
```

### 项目笔记模板

```markdown
# 项目名称 - 技术栈

## 架构设计
```
┌────────────┐
│  用户    │
└────┬─────┘
     │
     ▼
┌────────────┐
│  API     │
└────┬─────┘
     │
     ▼
┌────────────┐
│  DB     │
└────────────┘
```

## 技术选型
- **API 框架**: FastAPI（性能、自动文档）
- **数据库**: PostgreSQL（JSONB、扩展性）
- **缓存**: Redis（性能、灵活性）
- **搜索**: OpenSearch（混合搜索、可扩展）

## 关键决策
1. 为什么选择 FastAPI？
   - 性能是 Flask 的 2-3 倍
   - 自动生成 Swagger 文档
   - 原生异步支持

2. 为什么选择 PostgreSQL？
   - 支持 JSONB 存储半结构化数据
   - 强大的索引能力
   - ACID 事务保证

## 性能指标
- API 响应时间: < 100ms (p95)
- 搜索延迟: < 50ms (p95)
- 缓存命中率: > 80%
- 并发请求: 1000+ QPS

## 遇到的挑战
1. OpenSearch 内存不足
   - 解决：调整 JVM 参数
   - 结果：性能提升 50%

2. Redis 连接池耗尽
   - 解决：优化连接池配置
   - 结果：错误率下降 90%

## 改进方向
- [ ] 添加 GraphQL API
- [ ] 实现分布式缓存
- [ ] 引入消息队列
```

---

## 🎯 补充练习题

### Docker 练习题

**基础题：**
1. 什么是 Docker 镜像？什么是容器？
2. Dockerfile 中 `COPY` 和 `ADD` 的区别是什么？
3. 如何查看容器的日志？

**进阶题：**
1. 如何优化 Docker 镜像大小？
2. Docker Compose 的 `depends_on` 和 `healthcheck` 如何配合使用？
3. 如何在 Docker Compose 中实现零停机部署？

**实战题：**
1. 为 FastAPI 应用编写 Dockerfile
2. 使用 Docker Compose 编排 FastAPI + PostgreSQL + Redis
3. 实现服务的健康检查端点

### FastAPI 练习题

**基础题：**
1. 什么是依赖注入？有什么好处？
2. FastAPI 的 `app = FastAPI()` 和 `app = FastAPI(dependencies=[Depends(verify_token)])` 有什么区别？
3. 如何在 FastAPI 中处理 404 错误？

**进阶题：**
1. 如何实现自定义中间件？
2. FastAPI 的 `BackgroundTasks` 有什么用途？
3. 如何实现全局异常处理？

**实战题：**
1. 实现一个带有分页的列表 API
2. 实现一个文件上传 API（带进度条）
3. 实现 WebSocket 端点

### PostgreSQL 练习题

**基础题：**
1. 什么是事务？ACID 是什么意思？
2. `VARCHAR` 和 `TEXT` 有什么区别？
3. 什么是索引？为什么需要索引？

**进阶题：**
1. 什么是 JSONB？和 JSON 有什么区别？
2. 如何使用 GIN 索引？
3. 什么是连接池？如何优化连接池？

**实战题：**
1. 设计一个博客文章的数据模型
2. 编写一个复杂的查询（JOIN、子查询）
3. 实现全文搜索功能

---

## 总结

Week 1: Infrastructure Foundation 是整个 RAG 系统的基础。通过学习这一周的内容，你已经：

✅ 理解了生产级基础设施的重要性  
✅ 掌握了 Docker Compose 的服务编排  
✅ 学会了使用 FastAPI 构建高性能 API  
✅ 了解了 PostgreSQL、OpenSearch、Redis 等核心组件  
✅ 配置了 Apache Airflow 进行工作流管理  
✅ 集成了 Ollama 和 Langfuse  
✅ 建立了完整的开发和测试环境  

**下一周预告**: Week 2 将深入数据摄取管道，学习如何从 arXiv 自动获取和解析学术论文！

---

## 📞 支持与反馈

- **GitHub Issues**: https://github.com/jamwithai/production-agentic-rag-course/issues
- **博客评论**: https://jamwithai.substack.com/p/the-infrastructure-that-powers-rag
- **Telegram 社群**: （如果有）

---

**学习愉快！🚀**
