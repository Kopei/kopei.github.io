---
title: What is vector database?
comment: true
date: 2023-12-17 17:50:37
tags: vector database
---
# 什么是向量数据库 Vector Database?

> 本文将为您简要介绍向量数据库的定义，基础原理，应用和市面上已有的实现选型比较。通过阅读本文，读者将对向量数据库有基本的认识，对日后开发大模型 AI 相关应用有所帮助。

## 定义

向量数据库也叫矢量数据库，是一种以数学向量的形式存储数据集合的数据库。更通俗的说法，向量就是一个数字列表，例如：[12, 13, 19, 8, 9]。这些数字表示维度空间中的一个位置，代表在这个维度上的特征。就像行和列号表示电子表格中特定单元格一样（例如，“A10”表示 A 列 10 行）。向量数据库的应用是使机器学习模型更容易记住先前的输入，从而使机器学习能够用于支持搜索、推荐和内容生成等应用场景。向量数据可以基于相似性搜索进行识别，而不是精确匹配，使计算模型能够在上下文中理解数据。

### 什么是向量？

向量是一组有序的数值，表示在多维空间中的位置或方向。向量通常用一个列或行的数字集合来表示，这些数字按顺序排列。在机器学习中，向量可以表示诸如单词、图像、视频和音频之类的复杂对象，由机器学习（ML）模型生成。高维度的向量数据对于机器学习、自然语言处理（NLP）和其他人工智能任务至关重要。一些向量数据的例子包括：

- 文本：想象一下你上次与聊天机器人互动的情景。它们是如何理解自然语言的呢？它们依赖于可以表示单词、段落和整个文档的向量，这些向量是通过机器学习算法转换而来的。
- 图像：图像的像素可以用数字数据描述，并组合成构成该图像的高维向量。
- 语音/音频：与图像类似，声波也可以分解为数字数据，并表示为向量，从而实现声音识别等人工智能应用。

## 向量数据库的兴起

向量数据库的兴起主要源于大模型 embedding 的应用。`Transformer`作为当今大模型的基础架构，在数据输入时需要对输入做 embedding, 由于当时主要是处理文本，所以这个 embedding 要做的就是词嵌入（word embedding），把文本转化为向量。由于大模型使用海量数据，数据的维度一般大于 1000 以上，所以临时或永久存储和计算（检索）这些高维向量数据就成了一个难题，这也是向量数据库崛起的一个主要原因。另一个方面，将人的输入以向量的方式临时存储起来，也可以提高类 chatgpt 应用的性能，让人觉得 ai 是有记忆的，可以检索历史问过的相关问题。
想要自己试一试 embedding 的应用吗？可以试试 openai 的[embedding api](https://platform.openai.com/docs/guides/embeddings/what-are-embeddings), 通过这个 api, 你的一个文本输入将会转化为多维向量。

## 向量数据库的具体实现

关于向量数据库引擎盖下的核心实现和算法，比如如何做两个向量间的相关性搜索，各个算法实现的优劣，请移步这几个链接, 作者以视频的方式做了介绍，更加直观：

- [向量数据库技术鉴赏](https://www.bilibili.com/video/BV11a4y1c7SW/)
- [向量数据库的崛起](https://guangzhengli.com/blog/zh/vector-database/#%E5%90%91%E9%87%8F%E6%95%B0%E6%8D%AE%E5%BA%93%E7%9A%84%E5%B4%9B%E8%B5%B7)

## 使用向量数据库的好处

机器学习模型无法记住超出其训练范围的任何信息，它们必须在每次查询时重新获取上下文（这就是许多简单聊天机器人的工作方式）。
每次将查询的上下文传递给模型都非常缓慢，因为可能涉及大量数据；而且成本高昂，因为数据必须移动，计算能力必须反复用于让模型解析相同的数据。而且在实践中，大多数机器学习 API 可能受到它们一次能够接受多少数据的限制(所谓的 token 限制）。这就是向量数据库发挥作用的地方：数据集只需通过模型一次（或在数据变化时定期进行），模型对该数据的嵌入被存储在向量数据库中。
总结来说，使用向量数据库主要有如下好处:

- **速度与性能**：向量数据库使用各种索引技术实现更快的搜索。向量索引与诸如最近邻搜索等距离计算算法特别有助于在数百万甚至数十亿数据点中搜索相关结果，实现了优化的性能。
- **可扩展性**：向量数据库通过水平扩展存储和管理大量非结构化数据，可以在查询需求和数据量增加的情况下保持性能。
- **拥有成本 TOC**：向量数据库是训练基础模型的有价值的替代选择，无论是从头开始还是进行微调。这降低了基础模型推理的成本和速度。
- **灵活性**：无论是处理图像、视频还是其他多维数据，向量数据库都设计用于处理复杂性。由于具有从语义搜索到对话式人工智能应用等多种用例，向量数据库的使用可以定制以满足您的业务和人工智能要求。
- **LLM 的长期记忆**：组织可以从通用型模型（如 IBM watsonx.ai 的 Granite 系列模型、Meta 的 Llama-2 或 Google 的 Flan 模型）入手，然后将自己的数据提供给向量数据库，以增强与检索增强生成相关的模型和人工智能应用的输出。
- **数据管理组件**：向量数据库通常还提供内置功能，以便轻松更新和插入新的非结构化数据。

## 向量数据库的选型
我们对市面上有的向量数据库从扩展性，查询速度，搜索准确性，灵活性和可及性做一个表格，做相应的比较。
| Tool | Scalability | Query Speed | Search Accuracy | Flexibility | Persistence | Storage Location |
|-------------------------|-------------|-------------|------------------|-------------|-------------|-------------------|
| Chroma | High | High | High | High | Yes | Local/Cloud |
| DeepsetAI | High | High | High | High | Yes | Local/Cloud |
| Faiss | High | High | High | Medium | No | Local/Cloud |
| Milvus | High | High | High | High | Yes | Local/Cloud |
| pgvector | Medium | Medium | High | High | Yes | Local |
| Pinecone | High | High | High | High | Yes | Cloud |
| Supabase | High | High | High | High | Yes | Cloud |
| Qdrant | High | High | High | High | Yes | Local/Cloud |
| Vespa | High | High | High | High | Yes | Local/Cloud |
| Weaviate | High | High | High | High | Yes | Local/Cloud |
| DeepLake | High | High | High | High | Yes | Local/Cloud |
| LangChain VectorStore | High | High | High | High | Yes | Local/Cloud |
| Annoy | Medium | Medium | Medium | Medium | No | Local/Cloud |
| Elasticsearch | High | High | High | High | Yes | Local/Cloud |
| Hnswlib | High | High | High | High | No | Local/Cloud |
| NMSLIB | High | High | High | High | No | Local/Cloud |

### 主要推荐
#### Chroma
Chroma 是由[Chroma.ai](www.trychroma.com/)开发的开源向量数据库。它专注于可伸缩性，能够高效地存储和查询大规模向量数据集。Chroma 采用分布式架构，具有横向可伸缩性，可以处理海量的向量数据。它利用 Apache Cassandra 实现高可用性和容错性，确保数据的持久性和耐久性。
Chroma 的一个独特之处在于其灵活的索引系统。它支持多种索引策略，如近似最近邻（ANN）算法，例如 HNSW 和 IVFPQ，从而实现快速准确的相似性搜索。Chroma 还提供全面的 Python 和 RESTful API，便于与自然语言处理流水线轻松集成。凭借其对可伸缩性和速度的强调，Chroma 是对于需要高性能向量存储和检索的应用而言的绝佳选择。虽然它可能没有一些其他工具那么高的可伸缩性或先进的搜索算法，但对于小到中型项目或想要快速入门向量数据库的初学者来说，它是理想的选择。

#### Faiss
Faiss 标志，由 Facebook AI Research 开发，是一种广泛使用的向量数据库，以其高性能的相似性搜索能力而闻名。它提供了一系列针对高效检索最近邻居的优化索引方法，包括 IVF 和 HNSW。Faiss 还支持 GPU 加速，可以在大规模嵌入式数据上进行快速计算。
Faiss 的一个显著特点是其支持多索引搜索，这将不同的索引方法结合在一起，以提高搜索准确性和速度。此外，Faiss 提供了 Python 接口，便于与现有的自然语言处理流水线和框架集成。凭借其对搜索性能和多功能性的关注，Faiss 是在需要对大规模嵌入集合进行快速准确相似性搜索的项目中的首选选择。

#### PostgreSQL + pgvector
Pgvector 是用于 Postgres 的开源向量相似性搜索。Pgvector 有助于在流行的开源关系数据库 PostgreSQL 上构建向量数据库。它利用 PostgreSQL 扩展系统强大的索引功能，提供高效的向量嵌入存储和检索。Pgvector 支持 CPU 和 GPU 推理，实现高性能的向量操作。
Pgvector 的一个关键优势是它与更广泛的 PostgreSQL 生态系统的无缝集成。用户可以利用 PostgreSQL 的丰富功能，如 ACID 兼容性和对复杂查询的支持，同时受益于向量特定的操作。Pgvector 扩展了 SQL 语法以处理向量操作，并提供了一个用于简化集成的 Python 库。凭借其与 PostgreSQL 的兼容性和高效的向量存储，pgvector 是需要无缝 SQL 集成的自然语言处理应用的可靠选择。

## 小结
向量数据库从算法和技术层面来说并不是一个完全全新的领域，但由于LLM兴起，我们对其有了更多的重视，当前已有的产品可以说是百家齐鸣，本文也没有完全罗列其它传统数据库厂商如SQL, Redis, Mongo等对向量数据库的支持。希望通过这次GAI的浪潮，乘风破浪者们能够在不同领域找到更多的机会。


#### 推荐阅读

- [chroma](https://github.com/chroma-core/chroma)
- [Faiss](https://github.com/facebookresearch/faiss)
- [IBM vector database](https://www.ibm.com/topics/vector-database)

