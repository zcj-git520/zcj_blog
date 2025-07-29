---
title: "LangChain 自定义工具"
date: 2025-03-25T22:00:38+08:00
draft: false
image: 1.png
categories:
    - computer
    - AI
---

# LangChain 自定义工具

## 引言

在构建 LangChain 代理时，工具(Tools)是核心组件之一。工具允许代理与外部世界交互，执行特定任务。本文将深入探讨如何在 LangChain 中创建和使用自定义工具，包括内置工具的使用和自定义工具的多种实现方式。

## 工具的基本组成

每个工具都包含几个关键组件：

- **name** (str): 必需且必须在工具集中唯一
- **description** (str): 可选但建议提供，代理用它来决定如何使用工具
- **return_direct** (bool): 默认为 False
- **args_schema** (Pydantic BaseModel): 可选但建议提供，可用于验证参数或提供few-shot示例

## 内置工具的使用

LangChain 提供了许多内置工具，请访问[工具集成](http://www.aidoczh.com/langchain/v0.2/docs/integrations/tools/)查看可用工具列表，在使用第三方工具时，请确保您了解工具的工作原理、权限情况：

### 维基百科查询工具

```python
def get_wikipedia(query):
    api_wrapper = WikipediaAPIWrapper(top_k_results=1, doc_content_chars_max=100)
    tool = WikipediaQueryRun(api_wrapper=api_wrapper)
    result = tool.run(query)
    return result
```

### Google 文字转语音工具

```python
def google_tts(text):
    tts = GoogleCloudTextToSpeechTool()
    speech_file = tts.run(text)
    return speech_file
```

## 自定义工具的实现方式

### 1. 使用 @tool 装饰器

这是定义自定义工具最简单的方式：

```python
from langchain_core.tools import tool

@tool
def search_city_tool(city: str) -> str:
    """搜索城市信息"""
    url = f"https://api.songzixian.com/api/china-city?dataSource=LOCAL_CHINA_CITY&district={city}"
    response = requests.get(url)
    if response.status_code == 200:
        return response.json()
    else:
        raise ToolException(f"错误：{city}不存在，获取城市数据失败")
```

使用说明：

- 函数文档字符串将自动用作工具描述
- 类型提示用于参数验证
- 可以抛出 ToolException 来处理错误

### 2. 使用 StructuredTool.from_function

这种方式提供了更多配置选项：

```python
from langchain_core.tools import StructuredTool

def get_city_by_ip(ip: str):
    """根据IP地址获取城市信息"""
    url = f"https://whois.pconline.com.cn/ipJson.jsp?ip={ip}&json=true"
    response = requests.get(url)
    try:
        if response.status_code == 200:
            return response.json()
    except Exception as e:
        raise ToolException(f"错误：ip获取城市数据失败，{ip}")

def ip_to_city_tool(ip: str):
    calculator = StructuredTool.from_function(
        func=get_city_by_ip,  
        handle_tool_error="ip获取城市数据失败"
    )
    try:
        calculator.invoke({"ip":ip})
        return calculator
    except Exception as e:
        print(e)
        return ""
```

优势：

- 可以自定义错误处理
- 支持更复杂的参数配置
- 可以添加额外的元数据

## 工具的错误处理

良好的错误处理对工具至关重要：

```python
@tool
def search_weather_tool(city_code: str) -> str:
    """搜索天气信息"""
    url = f"http://t.weather.itboy.net/api/weather/city/{city_code}"
    response = requests.get(url)
    if response.status_code == 200:
        return response.json()
    else:
        raise ToolException(f"错误：{city_code}不存在，获取城市天气数据数据失败")
```

最佳实践：

- 使用 ToolException 抛出明确的错误
- 提供有用的错误信息
- 考虑添加重试逻辑

## 实际应用示例

### 城市信息查询系统

```python
# 获取城市代码
city_data = search_city_tool("北京")
city_code = city_data["code"]

# 获取天气信息
weather_data = search_weather_tool(city_code)

# 转换为语音
speech_file = google_tts(f"当前天气：{weather_data['weather']}")
```

## 工具的高级用法

1. **参数验证**：使用 Pydantic 模型验证输入参数
2. **Few-shot 示例**：在 args_schema 中添加示例
3. **工具组合**：将多个工具组合成工具包
4. **异步支持**：实现异步版本的工具

## 性能优化建议

1. 为网络请求添加超时设置
2. 实现缓存机制
3. 考虑批量处理请求
4. 监控工具使用情况

## 总结

LangChain 的工具系统提供了强大的扩展能力，允许开发者：

- 轻松集成现有API和服务
- 构建复杂的代理工作流
- 实现自定义业务逻辑
- 处理各种边缘情况

通过合理设计工具，可以大大增强代理的能力和可靠性

## 代码示例

```python
# 自定义工具调用
"""
当构建自己的代理时，您需要为其提供一组工具列表，代理可以使用这些工具。除了调用的实际函数之外，工具由几个组件组成：

name（str），是必需的，并且在提供给代理的工具集中必须是唯一的
description（str），是可选的但建议的，因为代理使用它来确定工具的使用方式
return_direct（bool），默认为 False
args_schema（Pydantic BaseModel），是可选的但建议的，可以用于提供更多信息（例如，few-shot 示例）或用于验证预期参数。
"""
import requests
from charset_normalizer.api import handler
from langchain_community.tools import WikipediaQueryRun
from langchain_community.utilities import WikipediaAPIWrapper
from langchain_community.tools import GoogleCloudTextToSpeechTool
from langchain_core.tools import tool, StructuredTool, ToolException

"""
内置工具包
"""
def get_wikipedia(query):
    api_wrapper = WikipediaAPIWrapper(top_k_results=1, doc_content_chars_max=100)
    tool = WikipediaQueryRun(api_wrapper=api_wrapper)
    result = tool.run(query)
    return result

def google_tts(text):
    tts = GoogleCloudTextToSpeechTool()
    speech_file = tts.run(text)
    return speech_file

"""
自定义工具调用
"""
# 使用@tool装饰器 -- 定义自定义工具的最简单方式。
@tool
def search_city_tool(city: str) -> str:
    """搜索城市信息"""
    url = f"https://api.songzixian.com/api/china-city?dataSource=LOCAL_CHINA_CITY&district={city}"
    response = requests.get(url)
    if response.status_code == 200:
        return response.json()
    else:
        raise ToolException(f"错误：{city}不存在，获取城市数据失败")

@tool
def search_weather_tool(city_code: str) -> str:
    """搜索天气信息"""
    url = f"http://t.weather.itboy.net/api/weather/city/{city_code}"
    response = requests.get(url)
    if response.status_code == 200:
        return response.json()
    else:
        raise ToolException(f"错误：{city_code}不存在，获取城市天气数据数据失败")
        # return ""
# StructuredTool.from_function 类方法提供了比@tool装饰器更多的可配置性，而无需太多额外的代码。

def get_city_by_ip(ip: str):
    """
    根据IP地址获取城市信息
    """
    url = f"https://whois.pconline.com.cn/ipJson.jsp?ip={ip}&json=true"
    response = requests.get(url)
    try:
        if response.status_code == 200:
            return response.json()
    except Exception as e:
        # print(e)
        raise ToolException(f"错误：ip获取城市数据失败，{ip}")

def ip_to_city_tool(ip: str):
    calculator = StructuredTool.from_function(func=get_city_by_ip,  handle_tool_error="ip获取城市数据失败")
    try:
        calculator.invoke({"ip":ip})
        return calculator
    except Exception as e:
        print(e)
        return ""



if __name__ == '__main__':
    print(search_city_tool.name)
    print(search_city_tool.description)
    print(search_city_tool.args_schema)
    print(search_city_tool("台北11111"))
    print(search_weather_tool("101260207"))

    # print(ip_to_city_tool("123.123.123"))
```

## 代码仓库路径

[zcj-git520/AiLargeModel: 大模型应用开发学习](https://github.com/zcj-git520/AiLargeModel)
