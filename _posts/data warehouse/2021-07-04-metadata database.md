---
layout: post
title:  Metadata in Data Warehouse
categories: [data warehouse]
description: 简介元数据库中的7种类型
keywords: metadata
catalog: true
multilingual: false
tags: metadata, data warehouse
---
## What is metadata?
`metadata`元数据是描述数据的数据。举个例子，一是原始单反拍摄的照片一般是RAW格式，这个格式的文件一般还是附带一些属性信息，如Etag, 描述照片拍摄的时间，照片拍摄的相机，曝光ISO, 分辨率等等信息，
这些信息就是照片的元数据。

## Metadata in data warehouse
在数据仓库中存在7种元数据:
- 数据定义和数据映射元数据，分别描述事实表和维度表的字段的意义和来源。数据映射元数据有时候也叫数据血缘（`data linage metadata`）, 数据血缘可以用于分析数据影响性分析。
- 数据结构元数据，描述每张表的表结构
- 数据源的元数据，描述原始数据库表结构和字段意义
- ETL元数据，描述每个ETL流程中的数据流
- 数据质量元数据，描述数据质量规则，对应的风险和措施
- 审计元数据，包含了所有数据仓库中的流程和活动
- 用量（`Usage metadata`）元数据，描述数据仓库的使用情况


