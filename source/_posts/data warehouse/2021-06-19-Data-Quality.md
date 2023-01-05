---
layout: post
title:  Data Quality Overview
categories: [data warehouse]
description: 数据治理如何保证?
keywords: data quality
catalog: true
multilingual: false
tags: data warehouse, data quality
updated: 2021-07-12
date: 2021-06-19
---

## The importance of data quality
数据质量对于数据报表至关重要, 数据的准确性(`Accuracy`), 完整性(`Completeness`), 一致性(`Consistency`), 精确性(`Precision`)和时效性(`timeliness`)这几个指标是评价数据质量的核心指标. 数据没有质量我们将构建数据仓库的意义将不复存在, 没有人相信一个数据不正确的数据仓库.

### Data cleaning & matching
数据清洗用于处理脏数据, 同时也用于识别相同的数据, 一般用到三种逻辑:
- exact
- fuzzy
- rule-based, 包括`incoming data, cross-reference, and internal rules`

### Action to violated data
当一个数据违反数据规则时, 我们有多种处理方案: 
- 拒绝数据进入仓库
- 允许数据进入仓库
- 修正数据

### DQ process
数据质量的控制一般分为三步: 
- 检查, 
- 报告,
- 修正.

如下图所示是一般的DQ流程.
![](/images/screenshots/screen_dq_process.png)