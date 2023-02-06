---
title: FHIR Rerource Definition
comment: true
date: 2023-01-31 11:18:44
tags: FHIR
---

## FHIR资源定义
本文主要记录FHIR的资源`Resource`是如何定义的。实际互操作时，资源可以用不同的格式来表示: `XML`、`JSON`、`Turtle`和`UML`. 未来批量数据的定义也发布`Bulk Data Formats`. 

### Resource Definition
资源可以用几种不同的方法来定义:
- 一个层级表格，以逻辑视图的方式展示内容, 简称逻辑表`Logical table`
- UML
- 片假XML语法
- 片假JSON
- 片假Turtle

除了上述语法，还有其他格式可以使用，包括`W3C schema`, `Schematron`, `JSON Schema`和`StructureDefinition`.

#### Logical table
逻辑表以树形结构来表示资源，并固定几个特定列。
