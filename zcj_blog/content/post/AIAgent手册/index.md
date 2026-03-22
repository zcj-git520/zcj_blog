---
title: "AI 智能体开发速查手册"
date: 2026-01-25T22:00:38+08:00
draft: false
image: 1.png
categories:
    - computer
    - AI
---

# AI 智能体开发速查手册

> 快速参考指南 - 常用代码、命令和配置

---

## 🚀 快速开始

### 环境搭建

```bash
# 创建虚拟环境
python -m venv venv
venv\Scripts\activate  # Windows
source venv/bin/activate  # Linux/Mac

# 安装核心依赖
pip install langchain langchain-community langchain-openai
pip install chromadb faiss-cpu
pip install fastapi uvicorn python-dotenv
pip install openai deepseek
```

### 基础项目结构

```
my-agent/
├── .env                    # 环境变量
├── requirements.txt        # 依赖
├── main.py                 # 入口
├── agent/
│   ├── __init__.py
│   ├── core.py             # 核心逻辑
│   ├── prompts.py          # 提示模板
│   └── tools.py            # 工具函数
└── knowledge/              # 知识库
```

---

## 📝 Prompt 模板

### 系统提示词模板

```python
SYSTEM_PROMPT = """你是一位{role}，具有以下特点：
- 专业领域：{expertise}
- 沟通风格：{style}
- 输出格式：{format}

约束条件：
1. {constraint1}
2. {constraint2}
3. 不知道的内容请诚实告知，不要编造

当前时间：{current_time}
"""
```

### Few-Shot 示例

```python
FEW_SHOT_EXAMPLES = """
用户：今天天气怎么样？
助手：我需要知道您所在的城市才能查询天气。请告诉我城市名称。

用户：北京天气
助手：北京今天晴，气温 15-25°C，空气质量良。

用户：上海明天会下雨吗？
助手：上海明天有小雨，气温 18-22°C，建议携带雨具。
"""
```

### CoT（思维链）模板

```python
COT_PROMPT = """请逐步思考这个问题：

1. 首先，理解问题的核心是什么
2. 然后，分析需要的信息和步骤
3. 接着，执行每个步骤
4. 最后，总结答案

问题：{question}

思考过程：
"""
```

---

## 🔧 LangChain 常用代码

### 基础 LLM 调用

```python
from langchain_openai import ChatOpenAI
from langchain_core.messages import HumanMessage, SystemMessage

llm = ChatOpenAI(
    model="gpt-4",
    temperature=0.7,
    api_key="your-key",
    base_url="https://api.openai.com/v1"
)

messages = [
    SystemMessage(content="你是一个有帮助的助手"),
    HumanMessage(content="你好")
]

response = llm.invoke(messages)
print(response.content)
```

### RAG 检索链

```python
from langchain_chroma import Chroma
from langchain_openai import OpenAIEmbeddings
from langchain.chains import create_retrieval_chain
from langchain.chains.combine_documents import create_stuff_documents_chain
from langchain_core.prompts import ChatPromptTemplate

# 向量存储
embeddings = OpenAIEmbeddings()
vectorstore = Chroma(
    collection_name="my_docs",
    embedding_function=embeddings,
    persist_directory="./chroma_db"
)

# 检索器
retriever = vectorstore.as_retriever(
    search_type="similarity_score_threshold",
    search_kwargs={"score_threshold": 0.7, "k": 3}
)

# 问答提示
qa_prompt = ChatPromptTemplate.from_template("""
基于以下上下文回答问题。如果不知道答案，请说不知道。

上下文：{context}

问题：{input}

答案：
""")

# 创建链
document_chain = create_stuff_documents_chain(llm, qa_prompt)
retrieval_chain = create_retrieval_chain(retriever, document_chain)

# 查询
result = retrieval_chain.invoke({"input": "你的问题"})
print(result["answer"])
```

### Agent 创建

```python
from langchain.agents import create_tool_calling_agent, AgentExecutor
from langchain_core.tools import Tool

# 定义工具
def search_web(query: str) -> str:
    """搜索互联网"""
    return f"搜索结果：{query}"

def calculator(expression: str) -> str:
    """计算数学表达式"""
    return str(eval(expression))

tools = [
    Tool(
        name="web_search",
        func=search_web,
        description="搜索互联网获取信息"
    ),
    Tool(
        name="calculator",
        func=calculator,
        description="执行数学计算"
    )
]

# 创建 Agent
prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个有帮助的助手，可以使用工具来回答问题"),
    ("human", "{input}"),
    ("placeholder", "{agent_scratchpad}")
])

agent = create_tool_calling_agent(llm, tools, prompt)
executor = AgentExecutor(
    agent=agent,
    tools=tools,
    verbose=True,
    handle_parsing_errors=True
)

# 执行
result = executor.invoke({"input": "计算 123*456 并搜索相关结果"})
```

---

## 🗄️ 向量数据库操作

### ChromaDB

```python
import chromadb
from chromadb.config import Settings

# 初始化客户端
client = chromadb.Client(Settings(
    persist_directory="./chroma_data",
    anonymized_telemetry=False
))

# 创建集合
collection = client.create_collection("documents")

# 添加文档
collection.add(
    documents=["文档内容 1", "文档内容 2"],
    metadatas=[{"source": "file1"}, {"source": "file2"}],
    ids=["doc1", "doc2"]
)

# 查询
results = collection.query(
    query_texts=["查询内容"],
    n_results=5,
    include=["documents", "metadatas", "distances"]
)
```

### FAISS

```python
from langchain.vectorstores import FAISS
from langchain_openai import OpenAIEmbeddings

embeddings = OpenAIEmbeddings()

# 从文档创建
vectorstore = FAISS.from_documents(documents, embeddings)

# 保存和加载
vectorstore.save_local("./faiss_index")
vectorstore = FAISS.load_local("./faiss_index", embeddings)

# 相似度搜索
docs = vectorstore.similarity_search("查询", k=5)
docs_score = vectorstore.similarity_search_with_score("查询", k=5)
```

---

## 🛠️ Function Calling

### OpenAI 格式

```python
from openai import OpenAI
import json

client = OpenAI(api_key="your-key")

tools = [
    {
        "type": "function",
        "function": {
            "name": "get_current_weather",
            "description": "获取当前天气",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {
                        "type": "string",
                        "description": "城市名称"
                    },
                    "unit": {
                        "type": "string",
                        "enum": ["celsius", "fahrenheit"]
                    }
                },
                "required": ["location"]
            }
        }
    }
]

response = client.chat.completions.create(
    model="gpt-4",
    messages=[{"role": "user", "content": "北京天气怎么样？"}],
    tools=tools,
    tool_choice="auto"
)

# 处理工具调用
message = response.choices[0].message
if message.tool_calls:
    for tool_call in message.tool_calls:
        func_name = tool_call.function.name
        args = json.loads(tool_call.function.arguments)
        # 执行函数
        result = eval(f"{func_name}(**{args})")
```

### 多轮工具调用

```python
def run_agent_with_tools(user_input, max_iterations=5):
    messages = [{"role": "user", "content": user_input}]
    
    for i in range(max_iterations):
        response = client.chat.completions.create(
            model="gpt-4",
            messages=messages,
            tools=tools
        )
        
        message = response.choices[0].message
        messages.append(message)
        
        if not message.tool_calls:
            return message.content
        
        # 执行工具调用
        for tool_call in message.tool_calls:
            result = execute_tool(tool_call)
            messages.append({
                "role": "tool",
                "tool_call_id": tool_call.id,
                "content": result
            })
    
    return "达到最大迭代次数"
```

---

## 📦 文档处理

### 加载器

```python
from langchain.document_loaders import (
    PyPDFLoader,
    Docx2txtLoader,
    TextLoader,
    UnstructuredMarkdownLoader,
    WebBaseLoader
)

# PDF
pdf_loader = PyPDFLoader("document.pdf")
pdf_docs = pdf_loader.load()

# Word
docx_loader = Docx2txtLoader("document.docx")
docx_docs = docx_loader.load()

# 网页
web_loader = WebBaseLoader("https://example.com")
web_docs = web_loader.load()

# 批量加载
from langchain.document_loaders import DirectoryLoader
dir_loader = DirectoryLoader("./docs", glob="**/*.pdf")
all_docs = dir_loader.load()
```

### 文本分块

```python
from langchain.text_splitter import (
    RecursiveCharacterTextSplitter,
    CharacterTextSplitter,
    TokenTextSplitter
)

# 递归分块（推荐）
splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,
    chunk_overlap=50,
    length_function=len,
    separators=["\n\n", "\n", "。", "！", "？", " ", ""]
)

chunks = splitter.split_documents(documents)

# Token 分块
token_splitter = TokenTextSplitter(
    chunk_size=512,
    chunk_overlap=50
)
```

---

## 🌐 API 部署

### FastAPI 示例

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from contextlib import asynccontextmanager

class QueryRequest(BaseModel):
    question: str
    top_k: int = 3

class QueryResponse(BaseModel):
    answer: str
    sources: list[str]

@asynccontextmanager
async def lifespan(app: FastAPI):
    # 启动时加载模型
    global agent
    agent = create_agent()
    yield
    # 关闭时清理

app = FastAPI(lifespan=lifespan)

@app.post("/query", response_model=QueryResponse)
async def query_endpoint(request: QueryRequest):
    try:
        result = agent.invoke({
            "input": request.question,
            "top_k": request.top_k
        })
        return QueryResponse(
            answer=result["answer"],
            sources=result["source_documents"]
        )
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

@app.get("/health")
async def health_check():
    return {"status": "healthy"}

# 运行
# uvicorn main:app --host 0.0.0.0 --port 8000
```

### Docker 配置

```dockerfile
# Dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  agent-api:
    build: .
    ports:
      - "8000:8000"
    environment:
      - OPENAI_API_KEY=${OPENAI_API_KEY}
    volumes:
      - ./knowledge:/app/knowledge
    restart: unless-stopped
  
  chromadb:
    image: chromadb/chroma
    ports:
      - "8001:8000"
    volumes:
      - chroma_data:/chroma

volumes:
  chroma_data:
```

---

## 🔍 调试技巧

### 启用详细日志

```python
import logging

logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger(__name__)

# LangChain 调试
from langchain.globals import set_debug, set_verbose
set_debug(True)
set_verbose(True)
```

### Token 计数

```python
import tiktoken

def count_tokens(text, model="gpt-4"):
    encoding = tiktoken.encoding_for_model(model)
    return len(encoding.encode(text))

# 计算对话 token
total_tokens = sum(
    count_tokens(msg["content"]) 
    for msg in messages
)
print(f"总 Token 数：{total_tokens}")
```

### 性能分析

```python
import time
from functools import wraps

def timing(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"{func.__name__} 耗时：{end-start:.2f}秒")
        return result
    return wrapper

@timing
def expensive_operation():
    # 你的代码
    pass
```

---

## 💰 成本优化

### 策略清单

1. **使用小模型处理简单任务**
2. **缓存常见查询结果**
3. **压缩 Prompt 长度**
4. **流式输出减少等待**
5. **批量处理请求**

### 缓存实现

```python
import hashlib
import json
from functools import lru_cache

def cache_key(messages):
    content = json.dumps(messages, sort_keys=True)
    return hashlib.md5(content.encode()).hexdigest()

@lru_cache(maxsize=1000)
def cached_llm_call(cache_key_str):
    # 从缓存获取或调用 LLM
    pass
```

---

## 📊 监控指标

### 关键指标

```python
from prometheus_client import Counter, Histogram, start_http_server

# 定义指标
REQUEST_COUNT = Counter('agent_requests_total', '总请求数')
REQUEST_LATENCY = Histogram('agent_request_latency_seconds', '请求延迟')
TOKEN_USAGE = Counter('agent_tokens_total', 'Token 使用量', ['type'])

# 记录
REQUEST_COUNT.inc()
with REQUEST_LATENCY.time():
    result = agent.invoke(input)
TOKEN_USAGE.labels(type='prompt').inc(prompt_tokens)
TOKEN_USAGE.labels(type='completion').inc(completion_tokens)
```

---

## 🎯 最佳实践

### Prompt 设计

✅ 好的做法：
- 明确角色和任务
- 提供具体示例
- 设定输出格式
- 包含约束条件

❌ 避免：
- 模糊的指令
- 过长的上下文
- 矛盾的要求
- 忽略边界情况

### 代码组织

```
# 推荐结构
agent/
├── config.py          # 配置
├── prompts.py         # 提示模板
├── tools.py           # 工具函数
├── chains.py          # 链定义
├── agents.py          # Agent 定义
└── utils/
    ├── embedding.py   # 嵌入相关
    ├── storage.py     # 存储相关
    └── monitoring.py  # 监控相关
```

### 错误处理

```python
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=4, max=10)
)
def call_llm_with_retry(messages):
    try:
        return llm.invoke(messages)
    except RateLimitError:
        logger.warning("速率限制，重试中...")
        raise
    except APIError as e:
        logger.error(f"API 错误：{e}")
        raise
```

---

## 📚 常用命令

```bash
# 检查 Python 版本
python --version

# 安装依赖
pip install -r requirements.txt

# 导出依赖
pip freeze > requirements.txt

# 运行 FastAPI
uvicorn main:app --reload

# Docker 操作
docker build -t my-agent .
docker-compose up -d
docker-compose logs -f

# 测试 API
curl -X POST http://localhost:8000/query \
  -H "Content-Type: application/json" \
  -d '{"question": "你好"}'
```

---

