---
layout: post
title: PowerBI Quickstart
categories: [Business Intelligence]
description: 
keywords: PowerBI
catalog: true
multilingual: false
tags: BI,powerbi
date: 2022-10-02
updated: 2022-10-04 18:30:00 
---

## PowerBI Quickstart Guide
截止2022年末市面上主流的BI产品有PowerBI, Tableau和国产帆软的FineBI. 由于Tableau逐渐退出中国，本文主要介绍一下PowerBI的基础功能。


### 数据导入方式
PowerBI支持从130+不同数据源直接导入数据，主要导入数据的方式分为:
- Direct import 直接导入
- DirectQuery 直接查询
- Live Connection 在线连接
  
这三种数据连接方式主要区别于:
- direct import 是将数据直接导进PowerBI desktop端的SSAS(SQL Server Analysis Services)的向量内存中，既然要存入内存那么对客户端的内存有一些要求，当然SSAS提供了一些内存存储压缩的算法来减少内存占用量。缺点是数据不是实时更新的。
- DirectQuery 顾名思义是直接在另一端的数据库上做查询, 这样性能的压力来到了数据库端。PowerBI目前支持主流的数据库，做查询前PowerBI desktop会将查询解释成对应的数据库语言到目标数据库执行，所以这里的语言转化或者叫做翻译是限制DirectQuery能力的瓶颈点。
- Live Connection只支持类SSAS的远程数据库，所有的能力和限制取决于远端SSAS的能力。

如下表可以用来决策什么时候用什么方式的数据连接，有时候我们可能需要一个综合的解决方案。
<img src="./images/screenshots/screen_shot_data_import.png" />


### 数据转化策略
PowerBI使用Power Query Editor来处理数据清洗和转换，底层使用的是`M`语言。在真正运行处理逻辑时，PowerQuery有个`Applied Steps`功能可以用来调整处理逻辑的顺序或者反悔某些操作如删除。

Power Query Editor在增加列的时候有一个自动根据所填例子形成新增列的功能，就是在新增一列时填入这列一行想转化成的数据，powerBI会自动完成剩余列的数据填充。

#### 条件列，向下填充，逆透视，合并查询和追加查询
`conditional column`就是在某列上经过if条件过滤后的生成一个新的列，比如某些列为空则生成新的列。
`fill down`向下填充就是把为空的值按最上面的非空值填充。
`split column`把一个列按一定规则拆成两个列。
`unpivot`逆透视是把原列头作为新的表的行值，而原来列的值将会被相应转成对应的行。
`merge queries`类似SQL的join, 分为内联，左联，右联，外联等。


### 数据建模
PowerBI有强大的数据建模能力，可以自动识别表的关系，处理多对多管理和角色扮演维表`role-play table`.

#### Role-play table
角色扮演维表是一张维表（比如时间维度表）对事实表有多个作用，同时一张表就可以扮演多个角色从而减少数据冗余。举个例子，一张销售事实表可有多个时间维度，如订单时间，交付时间，维护时间等。PowerBI需要使用非激活的关系和DAX度量来处理`Role-play`, 具体需要用到`USERELATIONSHIP`函数。