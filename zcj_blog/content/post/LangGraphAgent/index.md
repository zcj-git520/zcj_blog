---
title: "深入探索 LangGraph"
date: 2025-06-30T22:00:38+08:00
draft: false
image: 1.png
categories:
    - computer
    - AI
---

# 深入探索 LangGraph

在人工智能快速发展的今天，构建能够自主决策、灵活调用工具的智能代理成为了许多开发者的目标。LangGraph 作为一个强大的框架，为我们提供了构建这类代理的理想工具。本文将深入探讨 LangGraph 的核心概念、架构设计以及实际应用，帮助你快速掌握这一框架的使用。

## 什么是 LangGraph？

LangGraph 是一个基于图论的框架，用于构建具有复杂决策能力的智能代理。它允许开发者将代理的行为建模为一个状态图，其中包含节点（代表计算步骤）和边（代表节点间的连接）。这种结构使得代理能够根据当前状态做出决策，选择合适的工具，并在必要时调整其行为。

与传统的线性工作流不同，LangGraph 提供的图结构赋予了代理更强的灵活性和自主性。代理可以根据输入动态选择执行路径，多次调用不同工具，并在获得新信息时调整其策略。

## LangGraph 的核心概念

### 状态（State）

状态是 LangGraph 中最基础的概念，它表示应用程序在某一时刻的快照。状态通常定义为一个 TypedDict 或 Pydantic 模型，包含代理在执行过程中需要跟踪的所有信息。

```
class State(TypedDict):
    messages: Annotated[list, add_messages]
```

在这个例子中，我们定义了一个简单的状态，只包含一个 messages 字段，用于存储对话历史。Annotated[list, add_messages] 表示这个字段将使用 add_messages 归约器，这意味着新消息会被添加到列表中，而不是替换整个列表。

### 节点（Nodes）

节点是图中的基本单位，代表一个计算步骤或状态转换。每个节点都是一个函数，接收当前状态作为输入，执行某些操作，并返回更新后的状态。

```
def node(state: State):
    messages = state["messages"]
    response = model.qwen_llm().invoke(messages)
    return {"messages": [response]}
```

这个示例展示了一个简单的节点，它接收当前状态中的消息，调用语言模型生成响应，并将响应添加到消息列表中。

LangGraph 支持多种类型的节点：

- 计算节点：执行计算或转换操作

- 条件节点：根据状态条件执行不同路径

- 工具节点：调用外部工具（如 API、数据库等）

### 边（Edges）

边定义了图中节点之间的连接关系，控制着程序的执行流程。LangGraph 提供了多种类型的边：

1. **普通边**：直接从一个节点连接到另一个节点

```
graph_builder.add_edge("node_a", "node_b")
```

1. **条件边**：根据状态动态决定下一个节点

```
graph_builder.add_conditional_edges(
    "chatbot",
    tools_condition,
)
```

1. **入口点**：指定图的起始节点

```
graph_builder.add_edge(START, "chatbot")
```

### 归约器（Reducers）

归约器决定了如何将节点返回的部分状态更新合并到整体状态中。默认情况下，归约器会覆盖现有状态，但我们可以自定义归约器以实现更复杂的行为。

```
class State(TypedDict):
    foo: int
    bar: Annotated[list[str], add]  # 使用add归约器，将新元素添加到列表
    baz: Annotated[str, replace]    # 使用replace归约器，替换现有值
```

## 构建你的第一个 LangGraph 应用

让我们通过一个实际的例子来了解如何使用 LangGraph 构建一个简单的智能代理。这个代理能够进行对话，并在需要时调用维基百科和 IP 地址解析工具。

### 步骤 1：导入必要的库

```
from langchain_community.tools import WikipediaQueryRun
from langchain_community.utilities import WikipediaAPIWrapper
from langchain_core.messages import HumanMessage
from langgraph.checkpoint.memory import MemorySaver
from langgraph.constants import START
from langgraph.graph import StateGraph
from langgraph.graph.message import add_messages
from langgraph.prebuilt import ToolNode, tools_condition
from sqlalchemy.sql.annotation import Annotated
from typing_extensions import TypedDict
```

### 步骤 2：定义状态

```
class State(TypedDict):
    messages: Annotated[list, add_messages]
```

### 步骤 3：定义节点

```
def node(state: State):
    messages = state["messages"]
    response = model.qwen_llm().invoke(messages)
    return {"messages": [response]}
```

### 步骤 4：定义工具

```
def tools():
    api_wrapper = WikipediaAPIWrapper(top_k_results=1, doc_content_chars_max=100)
    tool = WikipediaQueryRun(api_wrapper=api_wrapper)
    return [tool, ip_to_city_tool]
```

### 步骤 5：构建图

```
def buildGraph():
    graph_builder = StateGraph(State)
    memory = MemorySaver()
    
    graph_builder.add_node("chatbot", node)
    tool_node = ToolNode(tools=tools())
    graph_builder.add_conditional_edges(
        "chatbot",
        tools_condition,
    )
    graph_builder.add_node("tools", tool_node)
    graph_builder.add_edge(START, "chatbot")

    graph = graph_builder.compile(checkpointer=memory)
    try:
        graph.get_graph().draw_mermaid_png(output_file_path="graph.png", max_retries=5, retry_delay=2.0)
    except Exception as e:
        print(f"渲染图表失败: {e}")
    return graph
```

### 步骤 6：运行对话

```
def chat(graph, msg):
    message = {"messages": [HumanMessage(content=msg)]}
    for event in graph.stream(message):
        for value in event.values():
            print("Assistant:", value["messages"][-1].content)
```

## 代理架构：超越简单路由

LangGraph 允许我们构建真正的智能代理，这些代理能够：

1. **在多条潜在路径之间进行路由**：根据输入和当前状态选择最佳执行路径

1. **决定调用众多工具中的哪一个**：根据任务需求选择合适的工具

1. **判断生成的答案是否足够**：决定是直接回答还是需要进一步的信息收集

这种架构超越了简单的路由机制，实现了真正的自主决策能力。

### ReAct 架构

ReAct 是一种流行的代理架构，它结合了三个核心概念：

- **工具调用**：允许 LLM 根据需要选择和使用各种工具

- **记忆**：使代理能够保留和使用来自先前步骤的信息

- **规划**：使 LLM 能够创建并遵循多步计划以实现目标

LangGraph 完美支持 ReAct 架构，通过状态管理实现记忆，通过条件边实现工具选择和规划。

## 高级特性

### 并行执行

LangGraph 支持节点的并行执行，这可以显著提高复杂工作流的效率。通过从一个节点扇出到多个节点，然后再扇入到一个节点，我们可以同时执行多个任务。

```
# 扇出到节点B和C，然后扇入到节点D
graph_builder.add_edge("A", "B")
graph_builder.add_edge("A", "C")
graph_builder.add_edge("B", "D")
graph_builder.add_edge("C", "D")
```

### 节点缓存

对于计算密集型节点，LangGraph 提供了缓存功能，可以避免重复计算：

```
from langgraph.pregel import CachePolicy

builder.add_node("expensive_node", expensive_node, cache_policy=CachePolicy(ttl=300))  # 缓存5分钟
graph = builder.compile(cache=InMemoryCache())
```

### 递归限制

为了防止无限循环，LangGraph 提供了递归限制功能，可以设置图在单次执行期间可以执行的最大步骤数：

```
graph.invoke(inputs, config={"recursion_limit": 10})  # 限制为10步
```

### 运行时配置

LangGraph 允许在运行时配置图的行为，例如指定使用哪个语言模型：

```
class ConfigSchema(TypedDict):
    llm: str

graph = StateGraph(State, config_schema=ConfigSchema)

# 调用时指定配置
graph.invoke(inputs, config={"configurable": {"llm": "gpt-4"}})
```

## 总结

LangGraph 为构建复杂的智能代理提供了强大而灵活的框架。通过将代理行为建模为状态图，我们可以创建能够自主决策、灵活调用工具的智能系统。无论是简单的对话代理还是复杂的多步骤工作流，LangGraph 都能提供清晰的结构和强大的功能支持。

通过本文的介绍，你应该对 LangGraph 的核心概念和使用方法有了基本的了解。接下来，不妨尝试使用 LangGraph 构建一个属于你自己的智能代理，探索更多高级特性和应用场景。

## 代码仓库路径

[zcj-git520/AiLargeModel: 大模型应用开发学习](https://github.com/zcj-git520/AiLargeModel)
