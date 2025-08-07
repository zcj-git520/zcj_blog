---
title: "LangGraph预构建Agent"
date: 2025-06-25T22:00:38+08:00
draft: false
image: 1.png
categories:
    - computer
    - AI
---

# LangGraph 预构建 Agent

智能代理（Agent）系统已成为连接用户与复杂功能的重要桥梁。LangGraph 作为一个强大的框架，为开发者提供了构建健壮、可投入生产的代理系统的丰富工具。

## 一、Agent 核心组件解析

LangGraph 预构建的 Agent 由三个核心部分构成，它们协同工作，使 Agent 能够理解用户需求、执行任务并返回结果：

1.**大型语言模型（LLM）**

这是 Agent 的 "大脑"，负责理解文本、生成响应并决定下一步行动。LLM 在一个循环中运行：每次迭代中选择工具、提供输入、接收结果（观察），并根据观察指导下一个动作，直到收集到足够信息响应用户。

 2.**工具（Tools）**

工具是 Agent 可以调用的函数或 API，用于执行特定任务。例如：

- - 网络搜索工具

- - 数据库查询工具

- - 第三方服务调用工具（如天气查询、IP 解析等）

工具使 Agent 能够突破自身知识局限，获取实时信息或执行复杂操作。

3.**提示（Prompt）**

提示是指导 LLM 行为的文本指令。它定义了 Agent 的角色、目标和行为准则。例如："你是一个智能小助手，能够使用工具帮助用户查询天气和城市信息"。

## 二、LangGraph 的核心优势

LangGraph 为构建 Agent 系统提供了多项关键功能，使其在实际生产环境中表现出色：

- **内存集成**：同时支持短期（会话内）和长期（跨会话）内存，实现有状态行为，让 Agent 能够记住历史对话和用户偏好。

- **人在回路控制**：允许执行过程在任何点暂停，等待人工反馈或审批，支持异步干预，这对于需要人工确认的敏感操作至关重要。

- **流式传输支持**：能够实时流式传输 Agent 状态、模型输出和工具结果，提升用户体验。

- **部署工具**：提供完整的测试、调试和部署工具链，简化从开发到生产的流程。

## 三、创建和配置 Agent

使用 LangGraph 创建预构建 Agent 非常简单，以下是基本步骤：

### 1. 定义工具

首先，需要定义 Agent 可以使用的工具。例如，我们可以创建 IP 解析、天气查询和城市搜索工具：

```python
tools = [
    ip_to_city_tool,  # 解析IP获取城市
    search_weather_tool,  # 查询天气
    search_city_tool  # 搜索城市信息
]
```

### 2. 创建提示

定义指导 Agent 行为的提示：

```python
from langchain_core.prompts import PromptTemplate

prompt = PromptTemplate(
    input_variables=[],
    template="你是一个智能小助手，能够帮助用户查询IP对应的城市以及城市的天气信息"
)
```

### 3. 初始化 Agent

使用 create_react_agent 函数创建 Agent：

```
from langgraph.prebuilt import create_react_agent

agent = create_react_agent(
    model=model.qwen_llm(),  # 配置的语言模型
    tools=tools,  # 上面定义的工具集
    prompt=prompt,  # 提示模板
    parallel_tool_calls=False  # 禁用并行工具调用
)
```

## 四、运行 Agent 的两种模式

LangGraph Agent 支持同步和异步两种运行模式，以适应不同的应用场景：

### 1. 同步模式

使用 .invoke() 或 .stream() 方法：

```
# 同步调用
result = agent.invoke({
    "messages": [HumanMessage(content="你好，请问杭州的天气如何？")]
})
print(result)
```

### 2. 异步模式

使用 await .ainvoke() 或 async for 与 .astream()：

```
# 异步调用
async for event in agent.astream({
    "messages": [HumanMessage(content="你好，请问杭州的天气如何？")]
}):
    print(event)
```

## 五、输入与输出格式

### 输入格式

LangGraph Agent 支持多种输入格式，提供灵活性：

- 字符串：{"messages": "Hello"}（解释为用户消息）

- 消息字典：{"messages": {"role": "user", "content": "Hello"}}

- 消息列表：{"messages": [{"role": "user", "content": "Hello"}]}

- 带自定义状态：{"messages": [...], "user_name": "Alice"}（适用于自定义状态）

### 输出格式

Agent 的输出是一个包含以下键的字典：

- messages：执行期间的所有消息（用户输入、助手回复、工具调用）

- structured_response（可选）：如果配置了结构化输出

- 自定义字段（可选）：基于自定义状态 schema 定义的额外键

## 六、控制 Agent 执行流程

为避免无限循环或失控执行，LangGraph 提供了多种控制机制：

### 1. 设置最大迭代次数

通过配置递归限制，定义 Agent 可执行的最大步骤数：

```
# 运行时配置
max_iterations = 3
recursion_limit = 2 * max_iterations + 1

result = agent.invoke(
    {"messages": [HumanMessage(content="Hello")]},
    {"recursion_limit": recursion_limit}
)
```

或在创建时配置：

```
agent = create_react_agent(
    llm=model.qwen_llm(), 
    tools=tools, 
    prompt=prompt
).with_config(recursion_limit=recursion_limit)
```

### 2. 工具调用控制

- **禁用并行工具调用**：parallel_tool_calls=False

- **直接返回工具结果**：在工具定义中使用 return_direct=True，使 Agent 在工具执行后立即停止

- **强制使用工具**：force_tool=True，确保 Agent 必须调用工具才能响应（需配合防护措施避免无限循环）

## 七、流式传输功能

流式传输是构建响应式应用的关键，LangGraph 支持多种流式传输类型：

### 1. Agent 进度流

获取每个节点执行后的更新：

```
for event in agent.stream(
    {"messages": [HumanMessage(content="你好")]}, 
    stream_mode="updates"
):
    print(event)
    print("="*20)
```

### 2. LLM 令牌流

实时获取语言模型生成的令牌：

```
for token, metadata in agent.stream(
    {"messages": [HumanMessage(content="获取这个ip：115.239.210.27的城市信息")]}, 
    stream_mode="messages"
):
    print("Token", token)
    print("Metadata", metadata)
```

## 八、上下文管理

Agent 通常需要除消息之外的更多信息才能有效工作，LangGraph 提供了三种上下文管理方式：

| 类型     | 描述                     | 可变性 | 生命周期       |
| -------- | ------------------------ | ------ | -------------- |
| 配置     | 运行开始时传入的静态数据 | ❌      | 每次运行       |
| 状态     | 执行中可更改的动态数据   | ✅      | 每次运行或对话 |
| 长期记忆 | 跨对话共享的数据         | ✅      | 跨对话         |

### 1. 配置（静态上下文）

在运行时传入不可变数据：

```
data = agent.invoke(
    {"messages":  [HumanMessage(content="获取这个城市信息")]},
    config={"configurable": {"ip": "115.239.210.27"}}
)
```

### 2. 状态（动态上下文）

定义自定义状态跟踪额外信息：

```
class CustomState(AgentState):
    user_name: str  # 跟踪用户名

agent = create_react_agent(
    # 其他参数...
    state_schema=CustomState,
)
```

### 3. 长期记忆（存储）

在对话之间持久化数据：

```
from langgraph.store.memory import InMemoryStore

# 初始化存储
store = InMemoryStore()

# 存储用户信息
store.put(
    ("users",),
    "user_123",
    {
        "name": "John Smith",
        "language": "English",
    }
)

# 创建使用存储的Agent
agent = create_react_agent(
    model=model.qwen_llm(),
    tools=tools,
    store=store
)
```

## 九、人机协作

LangGraph 内置了人机协作（HIL）功能，允许在 Agent 执行过程中加入人工审查步骤：

```
def book_hotel(hotel_name: str):
    # 暂停执行，等待人工审批
    response = interrupt(
        f"尝试调用 `book_hotel`，参数为 {{'hotel_name': {hotel_name}}}。请批准或建议修改。"
    )
    
    # 根据人工反馈继续执行
    if response["type"] == "accept":
        pass  # 执行预订
    elif response["type"] == "edit":
        hotel_name = response["args"]["hotel_name"]  # 使用修改后的参数
    else:
        raise ValueError(f"未知响应类型: {response['type']}")
        
    return f"成功预订 {hotel_name} 的住宿。"
```

这种机制使 Agent 能够在关键操作处暂停，等待人工确认，大大提高了系统的可靠性和安全性。

## 十、多智能体系统

对于复杂任务，单个 Agent 可能难以应对，此时可以构建多智能体系统。LangGraph 支持两种主要架构：

### 1. 主管架构

由一个中央主管 Agent 协调多个专业 Agent：

```
from langgraph_supervisor import create_supervisor

# 创建专业Agent
search_city_agent = create_react_agent(...)
weather_agent = create_react_agent(...)
ip_agent = create_react_agent(...)

# 创建主管Agent
supervisor = create_supervisor(
    model=model.qwen_llm(),
    agents=[search_city_agent, weather_agent, ip_agent],
    prompt="你是一个智能体协调器，根据用户问题调用合适的智能体"
).compile()
```

### 2. 群组架构

Agent 之间根据专业性动态移交控制权：

```
from langgraph.prebuilt import create_swarm

# 创建多个Agent
agent1 = create_react_agent(...)
agent2 = create_react_agent(...)
agent3 = create_react_agent(...)

# 创建群组
swarm = create_swarm(
    agents=[agent1, agent2, agent3],
    default_active_agent="agent1"  # 默认激活的Agent
).compile()
```

多智能体系统通过**移交（handoffs）** 机制实现通信，一个 Agent 可以将控制权移交给另一个 Agent，并传递必要的信息。

## 总结

LangGraph 预构建 Agent 为开发者提供了构建强大智能代理系统的便捷途径。通过其丰富的功能特性 —— 包括灵活的工具集成、完善的上下文管理、可靠的执行控制和多智能体协作能力 —— 开发者可以快速构建出适应各种场景的智能应用。

无论是简单的信息查询还是复杂的多步骤任务处理，LangGraph 都能提供坚实的技术支持，帮助你打造高效、可靠且用户友好的智能系统。

想要了解更多细节，可以访问 [LangGraph 官方文档](https://langgraph.com.cn/agents/overview.1.html)。

## 代码仓库路径

[zcj-git520/AiLargeModel: 大模型应用开发学习](https://github.com/zcj-git520/AiLargeModel)
