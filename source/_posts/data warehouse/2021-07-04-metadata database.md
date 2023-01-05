---
layout: post
title:  Metadata in Data Warehouse
categories: [data warehouse]
description: 简介元数据库中的7种类型
keywords: metadata
catalog: true
multilingual: false
tags: metadata, data warehouse
updated: 2021-07-13
date: 2021-07-13
---
## What is metadata?
`Metadata`元数据是描述数据的数据。举个例子，一是原始单反拍摄的照片一般是RAW格式，这个格式的文件会附带一些属性信息，如Etag, 描述照片拍摄的时间，拍摄的相机，曝光ISO, 分辨率等等信息，
这些信息就是照片的元数据。

## Why do we need metadata?
为什么需要元数据, 主要为了向用户解释数据和数据仓库, 让用户更好地理解数据仓库. 

## Metadata in data warehouse
在数据仓库中存在7种元数据:
- 数据结构元数据，描述每张表的表结构. 描述NDS, DDS, ODS和staging库中所有表结构, 包括`collation`(字符序). 这些信息大部分可以通过`object catalog view`拿到, 但是ETL元数据, DQ元数据, 数据定义元数据也需要结构信息, 所以我们需要单独创建这个库.
- 数据定义和数据映射元数据，分别描述事实表和维度表的字段意义和来源。为了避免混淆和误解, 必须有一个清晰, 全公司都能理解的字段定义. 数据映射元数据有时候也叫数据血缘（`data linage metadata`）, 数据血缘可以用于数据影响性分析。
- 数据源的元数据，描述原始数据库表结构和字段意义. 具体包含数据类型, 字符序, 主键, 外键, 视图, 索引和分区.
- ETL元数据，描述每个ETL流程中的每个数据流. 描述数据的流向, 所经过的转换, 父流程和定时任务.
- 数据质量元数据，描述数据质量规则，对应的风险和措施.
- 审计元数据，包含了所有数据仓库中的流程和活动.
- 用量（`Usage metadata`）元数据，描述数据仓库的使用情况.


