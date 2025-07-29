---
title: "LangChain 多模态输入处理"
date: 2025-02-25T22:00:38+08:00
draft: false
image: 1.png
categories:
    - computer
    - AI
---

# LangChain 多模态输入处理

## 引言

随着大模型技术的发展，多模态能力已成为AI应用的重要方向。LangChain提供了强大的多模态处理能力，允许开发者将文本、图像、PDF等多种输入类型结合使用。本文将深入探讨LangChain中的多模态输入处理机制。

## 多模态基础概念

多模态(Multimodality)是指模型能够同时处理和生成多种类型的数据，包括但不限于：
- 文本(Text)
- 图像(Image)
- 音频(Audio)
- 视频(Video)
- 文档(Document)

## LangChain多模态架构

LangChain的多模态处理主要基于以下组件：
- `HumanMessage`: 承载多模态内容的核心消息类
- 内容类型标识(type字段): 标识不同媒体类型
- 数据来源标识(source_type字段): 区分本地base64或URL

## 多模态输入实现方式

### 1. 基于Base64的本地文件输入

```python
def input_by_base64(self, prompt_txt, image_data, type):
    try:
        message = HumanMessage(
            content=json.dumps([
                {"type": "text", "text": prompt_txt},
                {"type": type, "data": image_data, "source_type": "base64"},
            ])
        )
        china = self.model.qwen_llm()
        return china.invoke([message]).content
    except Exception as e:
        print(e)
        return ""
```

使用场景：

- 处理本地存储的图片、PDF等文件
- 需要数据隐私保护的场景
- 离线应用环境

### 2. 基于URL的网络资源输入

```python
def input_by_url(self, prompt_txt, url, type):
    try:
        message = HumanMessage(
            content=json.dumps([
                {"type": "text", "text": prompt_txt},
                {"type": type, "data": url, "source_type": "url"},
            ])
        )
        china = self.model.qwen_llm()
        return china.invoke([message]).content
    except Exception as e:
        print(e)
        return ""
```

使用场景：

- 处理网络上的公开资源
- 需要动态加载内容的场景
- 减少本地存储压力的应用

## 文件类型支持

LangChain支持多种文件类型，常见的有：

| 类型     | MIME类型              | 描述               |
| :------- | :-------------------- | :----------------- |
| 图片     | image/png, image/jpeg | 支持常见图片格式   |
| PDF      | application/pdf       | 支持PDF文档解析    |
| 文本文件 | text/plain            | 纯文本文件         |
| Word文档 | application/msword    | Microsoft Word文档 |

## 实用工具函数

### 本地文件转Base64

```python
import base64

def file_to_base64_by_local(file_path):
    with open(file_path, "rb") as file:
        return base64.b64encode(file.read()).decode('utf-8')
```

### 图片处理工具

```python
from PIL import Image
import io

def process_image(image_data):
    # 从base64解码
    image_bytes = base64.b64decode(image_data)
    image = Image.open(io.BytesIO(image_bytes))
    # 进行各种图像处理...
```

## 实际应用案例

### 1. 图片内容描述

```python
image_path = "path/to/image.png"
base64_code = images_util.image_to_base64_by_local(image_path)
result = multimodal.input_by_base64(
    "这张图片描述了什么,中文描述", 
    base64_code, 
    "image/png"
)
```

### 2. PDF文档摘要

```python
file_path = "path/to/document.pdf"
base64_file = images_util.file_to_base64_by_local(file_path)
result = multimodal.input_by_base64(
    "文件主要讲了什么", 
    base64_file, 
    "application/pdf"
)
```

### 3. 网络图片分析

```python
result = multimodal.input_by_url(
    "这张图片描述了什么,中文描述", 
    "https://img.shetu66.com/2023/04/25/1682391094827084.png", 
    "image"
)
```

# 代码示例

```python
# 多模态输入
import base64
import json

from langchain_core.messages import HumanMessage

from common.modeCommon import Model
from config.config import Config
from utils import images_util


class Multimodal(object):
    def __init__(self, model: Model):
        self.model = model

    def input_by_base64(self, prompt_txt, image_data, type):
        try:

            message = HumanMessage(
                content=json.dumps([
                    {"type": "text", "text": prompt_txt},
                    {"type": type, "data": image_data, "source_type": "base64"},
                ])
            )

            china = self.model.qwen_llm()
            return china.invoke([message]).content
        except Exception as e:
            print(e)
            return ""

    def input_by_url(self, prompt_txt, url, type):

        try:
            message = HumanMessage(
                content=json.dumps([
                    {"type": "text", "text": prompt_txt},
                    {"type": type, "data": url, "source_type": "url"},
                ])
            )
            china = self.model.qwen_llm()
            return china.invoke([message]).content
        except Exception as e:
            print(e)
            return ""

if __name__ == '__main__':
    config = Config('conf/config.yml')
    # 初始化模型
    model = Model(config)
    multimodal = Multimodal(model)
    # image_path = "C:\\Users\\Administrator\\Desktop\\17536735231103.png"
    # base64_code = images_util.image_to_base64_by_local(image_path)

    # result = multimodal.input_by_image_base64("这张图片描述了什么,中文描述", base64_code, "image/png")
    # result = multimodal.input_by_url("这张图片描述了什么,中文描述", "https://img.shetu66.com/2023/04/25/1682391094827084.png", "image")
    file_path = "C:\\Users\\Administrator\\Desktop\\1.pdf"
    base64_file = images_util.file_to_base64_by_local(file_path)
    result = multimodal.input_by_base64("文件主要讲了什么", base64_file, "file")
    print(result)

```

## 代码仓库路径

[zcj-git520/AiLargeModel: 大模型应用开发学习](https://github.com/zcj-git520/AiLargeModel)