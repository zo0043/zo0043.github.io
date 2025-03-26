---
title: "‌‌‬​⁢⁡‍⁤⁤‬​‌‬⁡⁣⁣​‌⁣‬‬‍​﻿⁢​‍‍‬‬⁤‍​⁤⁤⁣⁢⁡​‬﻿‍‍⁢‌‍⁤​‌​09-dify案例分享-基于Gitee AI 在dify上实现生成毛茸茸风格图片 - 飞书云文档"
source: "https://aqma351r01f.feishu.cn/wiki/FolPwRC8HipgiNkI9MRcUqS3njh?table=tbleOWb4WgXcxiHK&view=vewGwwbpzl"
author:
published:
created: 2025-03-26
description:
tags:
  - "clippings"
---
## summary
本文介绍了Gitee AI和dify平台如何结合使用，通过Gitee AI的无服务器API和dify的LLM应用开发平台，实现了基于毛茸茸风格的图片生成案例。

## context
最近修改: 3月23日 09:19 共有 0 个协作者

## 09-dify案例分享-基于Gitee AI 在dify上实现生成毛茸茸风格图片

3月23日修改

本文讨论了基于Gitee AI在dify上实现生成毛茸茸风格图片的案例，介绍了Gitee AI和dify的相关概念、实现步骤及测试情况。关键要点包括： 1.Gitee AI ：一种无需管理服务器的API，为AI开发者提供大模型推理服务，采用Serverless架构，降低技术门槛和成本，有丰富功能层API。 2.dify ：开源的大语言模型应用开发平台，具有低代码/无代码开发、模块化设计等核心功能，支持多种大语言模型，面向不同技术背景开发者，有活跃社区支持。 3.实现步骤 ：先在Gitee平台开通申请Serverless API并获取令牌；根据接口文档开发Gitee AI接口，用多种编程语言示例代码，借助腾讯OSS存储图片；将接口部署到公网服务器或dify局域网内；在dify上进行工作流开发，包括开始节点、http请求、代码执行和直接回复等节点设置。 4.测试与总结 ：发布工作流并分享链接进行测试，可从本地上传图片或粘贴链接输入。Gitee平台Serverless API方便且模型多，推广期每天提供100次免费调用 。

1 什么是Gitee AI Gitee AI 是一种无需管理服务器的 API，旨在为 AI 开发者提供开箱即用的大模型推理服务。这种 API 允许开发者通过简单的注册和配置即可使用，无需关心底层基础设施的管理和维护，从而降低了技术门槛和成本。 Serverless 架构的核心思想是让开发者专注于业务逻辑，而不是底层的服务器管理。这种架构模式特别适用于需要快速扩展和按需付费的应用场景。在 Gitee 平台上，Serverless API 可以通过简单的操作创建和管理，支持多种功能层 API，如对话、文生图、语音识别等 Gitee AI地址 2 什么是dify Dify是一个开源的大语言模型（LLM）应用开发平台，旨在简化和加速生成式AI应用的创建和部署。它结合了后端即服务（Backend as Service, BaaS）和LLMOps的理念，使开发者能够快速搭建生产级的AI应用。 Dify的核心功能包括： 1.低代码/无代码开发 ：Dify提供了一个用户友好的界面，通过可视化的方式允许开发者轻松定义Prompt、上下文和插件等，无需深入底层技术细节。 2.模块化设计 ：采用模块化的设计，每个模块都有清晰的功能和接口，可以根据需求选择性地使用。 3.丰富的功能组件 ：包括AI工作流、RAG管道、Agent、模型管理、可观测性功能等，帮助开发者从原型到生产的全过程。 4.支持多种大语言模型 ：已支持OpenAI GPT系列等模型，并计划进一步扩展。 评论（0） 跳转至首条评论 0 字
