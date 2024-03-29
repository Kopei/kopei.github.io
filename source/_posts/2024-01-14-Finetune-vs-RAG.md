---
title: Finetune vs RAG
comment: true
date: 2024-01-14 15:27:22
tags: AI, RAG, Finetune
---

> 在生成式人工智能领域目前的争论之一是围绕微调、检索增强生成（RAG）和提示语（prompt）哪个是最佳方案，或者他们两两组合是不是合适的选择。在这篇文章中，我们将探讨RAG和Finetune两种技术，突显它们的优势、劣势以及可能的两者结合方案。通过阅读本文，您将清晰了解如何充分利用这些方法的潜力，推动您的人工智能取得成功。

## 两者不是对立关系
在深入比较之前，了解微调和检索增强生成并不是对立的技术是至关重要的。相反，它们可以结合使用，以充分发挥每种方法的优势。本文后面将举例说明如何让两者结合，提供企业级解决方案。如果不想看两者的简介，可以直接看下图比较

![ee](images/2024-01/ft-vs-rag.webp)

## 微调大模型Finetune简介
大多数当前的语言模型基于变种的Transformer，需要在大量文档的语料库上进行“预训练”，以学习单词之间的统计关系。语言模型学到的一切都存储在其基础人工神经网络的权重中。

微调就是在以后的某个日期对已经训练过的语言模型进行额外数据的训练，以便重新调整其神经网络的权重以适应新的输入。微调和预训练的关键区别在于，您对模型进行微调的数据通常比LM在初始训练期间摄取的一般数据更为具体，符合实际场景使用。

#### 微调的优势
微调允许语言模型专注于特定的领域或任务。例如，彭博（Bloomberg）在金融数据上训练了一个模型（BloombergGPT）；OpenAI对ChatGPT进行了问题-答案对的微调；ChatDoctor是一款使用LLaMA模型并结合医学知识进行微调训练的医疗助手。

在这里，可以突破您的想象力是限制，您可以在诗歌、表情符号、您自己的日记、替代文本、小语种或任何您能够想象到的基于语言的数据上进行微调。视觉、音频和多模态模型都适用于微调。

除了让语言模型专注于特定领域外，微调还享有附加优势，即允许使用更小的模型以及更便宜的训练和推理成本。例如，您通常可以使用大型预训练的语言模型，在其最初训练的数据的一小部分上进行微调，仍然能够在特定领域的应用中取得出色的性能。

#### 微调的劣势
微调语言模型虽然听起来像是一种灵丹妙药，但并非没有缺点。首先，尽管微调比从头开始训练语言模型更快，但仍然需要时间。其中之一是收集和准备用于微调的新数据。也许像谷歌这样的组织可以几乎每天都对LLM进行微调，但人类处理的许多信息甚至比日常数据更具临时性（例如，天气和地缘政治可能在一瞬间发生变化，当它们发生变化时，有些人需要更早地了解情况，而不是等待进行大模型微调所需的时间）。

除了不能充分解决大模型的时效性问题外，反复的微调可能会引发灾难性的遗忘。由于人工神经网络的权重存储了它们在训练期间学到的知识，而这些权重在微调期间稍后会更新，如果你对模型进行足够多次的微调，它可能会直接“忘记”一些先前的知识。当然，这会留下类似幻觉的问题。微调的大模型也可能“记忆”数据，这可能看起来不像是问题（肯定比忘记好），但是记忆可能引发隐私问题（如果训练数据清理得不够充分），而当LMs记忆数据时，它们对与其训练语料库差异显著的文本的泛化能力不强。

## RAG（Retrieval Augmented Generation）介绍
现在我们知道微调可以使大模型更加专注于更专业的领域，但我们不能仅仅为了使它们的知识及时而不断进行微调。那么，我们还能以什么其他方式解决大模型的时效性和幻觉问题呢？

如果大模型能够访问一系列外部文档，而不仅仅依赖于它们的内部知识，会怎么样呢？这基本上就是检索增强生成（RAG）所实现的概念，这个概念是在2020年Facebook的一篇论文中正式介绍的。随着LLM能力和受欢迎程度的不断提高，人们对它们的缺陷有了越来越多的认识，这促使对最初的RAG概念进行了实验。虽然不同的RAG变体提供了许多细节，但我们将专注于RAG的基本过程。

#### RAG的本质
RAG的本质是让模型获取正确的上下文，利用ICL (In Context Learning)的能力，生产足够多上下文的prompt输入以确保输出正确的响应。它综合利用了固化在模型权重中的参数化知识和存在外部存储中的非参数化知识(知识库、数据库等)。

首先，RAG类似于从图书馆（文档集合或数据库）借阅几本书，浏览它们的目录和索引，以判断它们是否可能包含你感兴趣的信息（检索），如果是的话，阅读相关的段落以帮助了解你想学习的内容（生成）。参数记忆是存储在人工神经网络内部的内在知识（即，大模型在不查阅外部来源的情况下已经知道的内容），而外部的非参数化知识存储在大模型权重之外（例如，在数据库中），但仍然可以被大模型访问。如果将原始的大模型检索类比为闭卷考试，而RAG则类似于开卷考试。

RAG如同名称所示，包含两个关键步骤：
1. 检索
2. 使用检索到的文档生成
首先，为了使RAG良好运作，大模型需要知道它们不知道的事。这对于RAG的检索步骤至关重要。例如，如果一个大模型过于依赖其内部的参数化知识，它很少会查阅任何内容，使得该大模型容易产生幻觉或像谄媚者一样回答。虽然我们希望LM通过RAG利用外部知识，但如果大模型经常查阅外部的非参数化知识，它们就会浪费内部的参数化知识，从而在不断的信息检索中陷入困境。想象一下，为了塑造你的每一个想法，你都要查阅一本书、文章或网站——没有人会这样做。相反，我们在使用我们内部深深根植的任何知识和我们能够找到的任何相关外部知识之间，取得了一种不完美但有用的平衡。理想情况下，RAG也应该做到这一点。
检索通常始于将文档、网站、数据库或其他数据嵌入到向量表示中，与大模型词嵌入（embedding）的方式有些相似，以便机器能够使用它们。对于安全性不太关心的应用，扩充数据可以是互联网（例如，Bing的生成搜索）。然而，注重安全性和准确性至关重要的应用（例如，企业应用）会将扩充数据限制在一个较小、安全且经过审查的池中。
接下来，用户的问题也被嵌入。然后，将用户嵌入的问题与嵌入的文档（或段落、句子等）进行比较。一些足够相似的外部知识然后通过一些提示词工程追加到用户的查询中。有了这个额外的上下文，大模型通常能够生成比没有外部信息更准确的响应。RAG方法通常将外部数据存储在矢量数据库中，因为矢量数据库有助于快速找到与查询相关的信息。
我们甚至可以设置流程来持续改进RAG。例如，可以将RAG生成的答案存储在与其他外部文档相同的矢量数据库中。然后，这些答案就像缓存的答案一样，因为当大模型在后来接收到足够相似的查询时，它可以检索其先前的答案（现在是一个嵌入的文档）并将其用作生成新答案的上下文。

![wkflow](images/2024-01/RAG_case.png)
RAG 工作流

#### RAG的优势
RAG具有许多语言模型的优势。首先，它在一定程度上解决了大模型的时效性问题，因为数据可以轻松而快速地添加到大模型可以访问的任何数据库中；这显然比定期对LM进行微调更快速、更经济。

其次，RAG显著减少了幻觉。即使使用RAG的大模型出现错误，RAG也允许大模型引用其信息来源，从而让我们能够识别、纠正或删除大模型检索到的不可靠信息。此外，由于RAG允许LM将其“知识”的大部分存储在外部来源中，您可以减少一些伴随着将敏感数据存储在大模型权重内的安全泄漏问题。此外，能够访问大型数据集的能力在一定程度上解决了大模型的上下文窗口限制问题，而且总体而言，RAG比持续微调更经济，当然这还要取决于需要为RAG嵌入多少文档以及某些微调语料库有多大。

#### RAG的劣势
RAG的劣势主要来自具体检索的实现上。这在实际工程实践上很有挑战。正如我们之前提到的，研究人员仍在努力找出如何让大模型可靠地知道何时查阅外部的非参数化知识以及何时依赖于其内部的参数化知识。查找字面所有信息的速度太慢且降低了大模型的效用，因此我们不希望RAG在检索部分过于宽泛。我们也不希望大模型总是凭直觉行事。找到一个平衡点可能是RAG最大的挑战。
还有更多的考虑。如果一个大模型决定查阅外部来源，它应该如何做呢？大模型应该考虑每个文档吗？还是应该将每个文档分成页面、段落或句子，以便大模型可以查询更精细的信息？同样，我们需要一个平衡点。这个决定影响如何对大模型的外部知识库进行矢量化。
同样，模型应该查阅哪些来源？整个互联网，还是其中的一些子集？手工制作的一组本地存储的文档？而RAG应该检索多少文档呢？所有相关文档吗？
通常，RAG检索前n个文档（其中n是某个自然数），这似乎是合理的，但如果回答查询所需的知识要点位于第n+1个文档中呢？当前的RAG方法通常使用近似最近邻方法（因为纯最近邻可能导致查阅太多文档），但近似最近邻不能保证为某个查询返回一个有用的文档，因此即使使用了ANN减轻了工作量，幻觉问题仍然存在。
最后，与仅查询大模型的内部知识相比，RAG可能更昂贵，因为RAG需要嵌入和存储大模型可以查阅的外部数据，通常会产生矢量数据库存储成本（但RAG仍然比例常规微调更便宜），并且RAG可能增加推理成本，因为它会将检索到的文档附加到查询中。当然，所有这些决策和考虑都依赖于具体的应用场景，但你可以看到实施RAG可能会变得多么复杂。

#### FT和RAG结合
RAG和微调为使大模型具有实时性并减少幻觉等问题行为提供了互补的优势和劣势。微调有助于使大模型专注于特定领域、词汇或新颖数据，但需要时间和计算资源，存在灾难性遗忘的风险，并且无法解决互联网快速演变的特性。RAG利用外部知识以增加事实和时效性，但仍然面临有关最佳检索的挑战。
两种方法都不能完全解决LM的所有缺陷，但都有助于解决一些即时的问题。随着LLMs不断渗透到我们的数字体验中，我们可以预期进一步尝试将微调、RAG和其他信息检索技术结合起来，为LLMs提供更及时、真实的知识。目前meta的技术人员就开发了一个模型RA-DIT, 知识密集型的零和少量次学习基准范围内实现了最先进的性能。

![images/2024-01/RAG_FT_eng.png](images/2024-01/RAG_FT_eng.png)
RAG和其它优化方式比较

#### 可能的企业级设计
如下图所示, 一个企业级的应用可以结合微调模型和RAG技术，整合内部其它业务系统和数据将内部知识安全地与外部知识融为一体，从而减轻企业的成本，提高运营的效率。比如使用下面的架构重新整合企业内部HR相关服务，提高HR Share Serivce Center的能力和效率。
![example](images/2024-01/example_arch.webp)

又如，想象一个客户支持聊天助手。当代理需要在实时互动中获得指导时，他们会提示助手，触发以下流程：
- 流水线检索关键客户信息，包括客户ID。
- 流水线查询客户的历史记录、问题、政策、特殊情况（例如故障）、当前支持团队的可用性以及其他动态外部变量。
- 流水线将收集到的上下文和原始查询成为最终提示。
- 模型生成一个基于提供的上下文的响应。
尽管此流水线可以使用“现成的”模型，但生成的响应的风格可能会有所不同，并偏离内部政策或最终用户的要求。为了提高一致性和帮助性响应的可能性，数据团队可以训练一个“专业化”的助手模型。
数据团队将从适当的基础模型开始，例如Llama 2或GPT-4。然后，他们将构建或策划一个例子语料库，其中包含模型将经常遇到的每种特定任务类型的可能输入（上下文+提示）和理想输出（包括期望的格式）。这可能包括针对客户服务代理自己处理的任务（例如重新激活帐户）以及涉及引导用户执行他们自己操作的任务（例如故障排除）的不同方法。
在这种情况下，微调和检索增强共同合作，以更快地提供正确的解决方案。

### 推荐阅读：
- [Retrieval-Augmented Generation for Large Language Models: A Survey](https://github.com/Tongji-KGLLM/RAG-Survey)
- [Retrieval-Augmented Dual Instruction Tuning](https://arxiv.org/abs/2310.01352)
- [RAG vs. Finetuning: Enhancing LLMs with new knowledge | Deepgram](https://deepgram.com/learn/rag-vs-finetuning)