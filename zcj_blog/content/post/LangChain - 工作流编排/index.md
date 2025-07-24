---
title: "LCEL与LangChain流式处理指南"
date: 2025-01-25T22:00:38+08:00
draft: false
image: 1.png
categories:
    - computer
    - AI
---

# LCEL与LangChain流式处理指南

## 什么是LCEL？

LCEL（LangChain Expression Language）是一种强大的工作流编排工具，能够从基本组件构建复杂任务链条(chain)，并具有以下生产级特性：

- 🚀 **一流的流式支持**：直接从LLM流式传输标记到输出解析器
- ⚡ **异步支持**：同一代码可同步/异步运行，适合原型到生产
- 🔀 **优化的并行执行**：自动并行可并行步骤
- 🔄 **重试和回退**：配置可靠性机制
- 🔍 **中间结果访问**：实时监控和调试
- 📐 **输入输出模式**：自动生成Pydantic/JSONSchema验证

## Runnable接口标准

LangChain组件通过Runnable协议实现标准化调用：

### 同步方法
- `stream()`: 流式返回响应块
- `invoke()`: 调用链处理输入
- `batch()`: 批量处理输入列表

### 异步方法
- `astream()`: 异步流式响应
- `ainvoke()`: 异步调用
- `abatch()`: 异步批量处理
- `astream_log()`: 流式中间步骤+最终结果
- `astream_events()`: 流式链事件(Beta)

### 组件I/O类型
| 组件       | 输入类型               | 输出类型   |
| ---------- | ---------------------- | ---------- |
| 提示       | 字典                   | 提示值     |
| 聊天模型   | 字符串/消息列表/提示值 | 聊天消息   |
| LLM        | 字符串/消息列表/提示值 | 字符串     |
| 输出解析器 | LLM/聊天模型输出       | 解析器特定 |
| 检索器     | 字符串                 | 文档列表   |
| 工具       | 字符串/字典            | 工具特定   |

## 流式处理(Stream)

### 核心方法
1. **同步流式**：`stream()`
2. **异步流式**：`astream()`

### 关键特性
- 实时返回每个处理块
- 要求所有步骤支持流式处理
- 复杂度范围：
  - 简单：LLM令牌流
  - 复杂：部分JSON结果流

### 最佳实践
建议从LLM组件开始逐步构建流式处理链。

## 事件流(Stream Events)

### 高级方法
- `astream_events()` (Beta): 流式传输链执行过程中的所有事件
- `astream_log()`: 流式中间步骤+最终输出

### 典型应用场景
1. 实时用户反馈
2. 复杂链调试
3. 进度监控
4. 增量结果展示

# 代码示例

```python
import asyncio
import logging

from langchain.chains.llm import LLMChain
from langchain_core.output_parsers import StrOutputParser, JsonOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.tools import tool

from langchain.chains import SimpleSequentialChain

import PromptStudy
from common.modeCommon import Model
from config.config import Config


# LangChain工作流编排

# 同步Stream运行
def sync_stream(model: Model, constr):
    for ch in model.qwen_llm_stream().stream(constr):
        print(ch.content, end='|', flush=True)


# @tool
# 异步AStream运行
async def async_stream(model: Model, constr):
    async for ch in model.qwen_llm_stream().astream(constr):
        print(ch.content, end='|', flush=True)


# Chain
async def llm_chain(model: Model, prompt: PromptStudy):
    # 创建一个工作链
    base_prompt = prompt.base_prompt()
    parser = StrOutputParser()
    stream_model = model.qwen_llm_stream()
    chain = base_prompt | stream_model | parser # 创建一个创建一个工作链 先执行base_prompt 然后执行stream_model 最后执行parser
    # 运行工作链
    async for ch in chain.astream(prompt.base_prompt_invoke("系统运维", "运维高级工程师", "想理解交换机原理", "无", "简短文字输出")):
        print(ch, end='|', flush=True)
# 序列链（Sequential Chains）
"""
    序列链是一种将多个链按照顺序连接在一起的工作流。
    每个链都可以接收前一个链的输出作为输入，并将其传递给下一个链。
    序列链可以用于构建复杂的工作流，其中每个链都有特定的任务和逻辑。
    序列链的主要优点是可以将多个链组合在一起，形成一个完整的工作流。
    序列链的主要缺点是每个链都需要独立运行，因此可能会导致性能问题。
"""
def one_chain(model: Model, prompt: PromptStudy):
    base_prompt = prompt.base_prompt()
    parser = JsonOutputParser()
    stream_model = model.qwen_llm_stream()
    chain = base_prompt | stream_model | parser
    return chain
def two_chain(model: Model, prompt: PromptStudy):
    base_prompt = prompt.base_prompt()
    parser = StrOutputParser()
    stream_model = model.qwen_llm_stream()
    chain = base_prompt | stream_model | parser
    return chain
def llm_chain_sequential(model: Model, prompt: PromptStudy):
    chain_one = one_chain(model, prompt)
    chain_two = two_chain(model, prompt)
    overall_chain = chain_one | chain_two
    # 运行工作链
    # 将 prompt.base_prompt_invoke 的返回值包装成字典
    # input_dict =prompt.base_prompt_invoke("系统运维", "运维高级工程师", "想理解交换机原理", "无", "简短文字输出")
    # 运行工作链
    input_dict = {
        "domain": "高铁",
        "profession": "自愿者",
        "do": "去哪里购买高铁票，我应该去找谁",
        "material": "无",
        "question": "按照以下json的格式输出：{"
                    "\"domain\": \"\","
                    "\"profession\": \"\","
                    "\"do\": \"\","
                    "\"material\": \"\", "
                    "\"question\": \"\"}"
    }
    for ch in overall_chain.stream(input_dict):
        print(ch, end='|', flush=True)

#转换链（Transform Chains）： 允许你在 链 的中间步骤中对数据进行转换
def transform_chain(model: Model):
    first_prompt = ChatPromptTemplate.from_template("描述一家生产{产品}的公司最好的名字是什么？")
    chain_one = model.qwen_llm_china(first_prompt)
    second_prompt = ChatPromptTemplate.from_template("为以下公司写一个20字的描述：{company_name}")
    chain_two = model.qwen_llm_china(second_prompt)
    overall_simple_chain = SimpleSequentialChain(chains=[chain_one, chain_two], verbose=True)
    product = "外卖和电商"
    chain_output = overall_simple_chain.invoke(product)
    print(chain_output)
#条件链（Conditional Chains）： 允许你根据某些条件选择不同的链来执行
#路由链（Router Chains）： 允许你根据某些条件选择不同的链来执行
#多模型链（Multi Model Chains）： 允许你在不同的模型之间切换



if __name__ == '__main__':
    logging.basicConfig(level=logging.INFO)
    # 初始化模型配置
    config = Config('conf/config.yml')
    # 初始化模型
    model = Model(config)
    # 初始化模板
    prompt = PromptStudy.Prompt()
    # sync_stream(model, "贵州省的组成")
    # asyncio.run(async_stream(model, "上海怎么样"))
    # asyncio.run(llm_chain(model, prompt))
    # llm_chain_sequential(model, prompt)
    transform_chain(model)
```

## 代码仓库路径

[zcj-git520/AiLargeModel: 大模型应用开发学习](https://github.com/zcj-git520/AiLargeModel)
