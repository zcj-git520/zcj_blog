---
title: "LangServe"
date: 2025-01-25T22:00:38+08:00
draft: false
image: 1.png
categories:
    - computer
    - AI
---

# LangServe 实践

## 核心功能
LangServe 是一个专为 LangChain 设计的部署工具，可将 LangChain 可运行对象和链快速转化为 REST API，主要特点包括：

- **自动化 API 生成**  
  通过 FastAPI + Pydantic 自动推断输入/输出模型，生成带数据验证的 REST 端点
- **多协议支持**  
  提供 `/invoke`（单次调用）、`/batch`（批量处理）、`/stream`（实时流）等标准化端点
- **调试友好**  
  - `/stream_log` 实时流式传输中间步骤
  - 0.0.40+ 版本新增 `/stream_events` 简化流式处理
- **生产级架构**  
  基于 uvloop 和 asyncio 实现高并发，支持 Swagger 自动文档

## 技术优势
| 特性           | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| **声明式验证** | 通过 Pydantic 模型自动生成 JSON Schema，提供类型检查和清晰错误提示 |
| **无缝集成**   | 内置 LangChain.js 客户端，支持 Python/JavaScript 调用        |
| **零模板代码** | 自动生成 API 文档和 Playground 调试界面                      |

## 典型工作流
**定义 LangChain 对象**  

```python
from langchain.chains import LLMChain
chain = LLMChain(...)
```

**部署为 API**

```
from langserve import add_routes
add_routes(app, chain)
```

**即时获得**：

- ✅ 自动生成的 Swagger 文档
- ✅ 带类型校验的 `/invoke` 端点
- ✅ 可视化调试 Playground

## 适用场景

- 🔧 快速原型验证
- 🚀 生产环境服务部署
- 🔗 微服务架构集成
- 🧪 交互式调试测试

## 代码示例

- 服务端

```python
import uvicorn
from fastapi import FastAPI
from fastapi.responses import RedirectResponse
from langchain_core.runnables import RunnableLambda
from langserve import add_routes

from Prompt import Prompt
from common.modeCommon import Model
from config.config import Config

app = FastAPI(title="LangChain 服务器",
              version="1.0",
              description="使用 Langchain 的 Runnable 接口的简单 API 服务器")


@app.get("/")
async def redirect_root_to_docs():
    return RedirectResponse("/docs")


def base_prompt_routes(prompt, model):
    base_prompt = prompt.base_prompt()
    llm_chain = model.qwen_llm_china(base_prompt)
    poetry_func = RunnableLambda(lambda x: llm_chain.invoke(x))
    add_routes(app, poetry_func, path="/base")

def chat_prompt_routes(prompt, model):
    chat_prompt = prompt.chat_prompt()
    llm_chain = model.qwen_llm_china(chat_prompt)
    poetry_func = RunnableLambda(lambda x: llm_chain.invoke(x))
    add_routes(app, poetry_func, path="/chat")

def stream_prompt_routes(prompt, model):
    llm_chain = model.qwen_llm_stream()
    poetry_func = RunnableLambda(lambda x: llm_chain.invoke(x))
    add_routes(app, poetry_func, path="/stream")

if __name__ == "__main__":
    # 初始化模型配置
    config = Config('conf/config.yml')
    # 初始化模型
    model = Model(config)
    # # 调用qwen模型
    prompt = Prompt()
    base_prompt_routes(prompt, model)
    chat_prompt_routes(prompt, model)
    stream_prompt_routes(prompt, model)
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

- 客户端示例

```python
from langserve import RemoteRunnable

remote_chain = RemoteRunnable("http://127.0.0.1:8000/base")
response = remote_chain.invoke({
            "domain": "金融",
            "profession": "金融分析员",
            "do": "评估未来半年中国的金融市场情况，分析市场趋势，预测未来的市场变化",
            "material": "无",
            "question": "以表格的形式输出",
        })

if __name__ == '__main__':
    print(response)
```

## 代码仓库路径

[zcj-git520/AiLargeModel: 大模型应用开发学习](https://github.com/zcj-git520/AiLargeModel)
