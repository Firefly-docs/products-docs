# RAG 私有知识库

检索增强生成（RAG）是一种使用来自私有或专有数据源的信息来辅助文本生成的技术，该技术通过结合大型语言模型（LLM）的生成能力与从外部知识库中检索相关信息的能力，来生成更加精准和上下文相关的回答或文本内容。

通过借助 RAG 技术，可以解决大模型存在的知识时效性不足、上下文理解限制、信息来源不确定等关键性问题。

本章节提供三个可部署于 AIO-1684XJD4 上的 RAG 部署案例，分别为 FireflyChat、ChatDoc-TPU 和 LangChain-Chatchat-TPU。

## FireflyChat 项目简介

FireflyChat 是由 Firefly 开源团队开发的图形化大模型应用平台，部署仅需简单安装、无需编译，快速体验 RAG 对大模型的提升。

项目详情参考 [FireflyChat](fireflychat.md) 章节。

## ChatDoc-TPU 项目简介

ChatDoc-TPU 是一个完全本地化推理的文档对话工具，其主要目标是通过使用自然语言来简化与文档的交互，并提取有价值的信息。

项目仓库链接：[ChatDoc-TPU](https://github.com/wangyifan2018/ChatDoc-TPU)

## LangChain-Chatchat-TPU 项目简介

Langchain-Chatchat-TPU 是基于 Langchain-Chatchat 开发的完全本地化推理的知识库增强方案。

项目仓库链接：[LangChain-Chatchat-TPU](https://github.com/wangyifan2018/LangChain-Chatchat-TPU)