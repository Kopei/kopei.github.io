---
layout: post
title:  Three type of data model 
categories: [data warehouse]
description: 三种数据模型
keywords: data modeling
catalog: true
multilingual: false
tags: data warehouse, data model
date: 2021-08-09
---

## Three Types of Data Models
一般主要有三种数据模型[数据模型定义](https://dl.acm.org/doi/book/10.5555/1594814):
- 概念模型
- 逻辑模型 
- 物理模型

这三种模型可以使用工具来建模[工具列表](https://en.wikipedia.org/wiki/Comparison_of_data_modeling_tools). 我们通常先创建概念数据模型，然后再做逻辑数据模型。

### 概念数据模型
概念数据模型用于定义高层业务抽象和概念。通常是业务所有者绘制，建模时不用考虑具体系统的约束。使用SQL Server的SSMS可以很好地建模，一般使用SSMS创建概念模型的流程是：
1. 创建数据库关系图
2. 在可编程性栏目下， 创建用户定义数据类型(可选)
3. 在关系图画布下，创建只包含主键的维表和事实表。
4. 使用拖拉关联事实表和维表外键关系
5. 自动调整表大小和自动布局

### 逻辑数据模型
逻辑数据模型用于指定实体的所有属性，并且识别实体之间的关系。逻辑模型通常是数据架构师定义的，用于业务分析。

### 物理数据模型
物理数据模型是把具体的逻辑数据模型使用某个数据库去具体实现。数据库开发者通常使用这个物理模型去做具体的开发工作。
