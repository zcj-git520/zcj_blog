---
title: "多模态技术指南：原理与应用实践"
date: 2025-02-25T22:00:38+08:00
draft: false
image: 1.png
categories:
    - computer
    - AI
---

# 多模态技术指南：原理与应用实践

## 1. 多模态技术概述

多模态技术是指系统能够处理和整合多种形式数据（如文本、音频、图像、视频等）的能力。这项技术正在重塑人机交互方式，使AI系统能够像人类一样理解复杂的世界。

### 核心组件支持
- **聊天模型**：处理多模态输入/输出
- **嵌入模型**：多模态数据向量化表示
- **向量存储**：跨模态数据检索

## 2. 聊天模型中的多模态实现

### 2.1 基础配置要求
- 支持多模态的聊天模型（如GPT-4 Vision、Gemini等）
- 消息内容块(content blocks)构造能力
- 供应商特定的API接入权限

### 2.2 输入处理实践

#### 通过URL传递图像
```python
from langchain_core.messages import HumanMessage

 message = HumanMessage(
                content=json.dumps([
                    {"type": "text", "text": prompt_txt},
                    {"type": type, "data": image_data, "source_type": "base64"},
                ])
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