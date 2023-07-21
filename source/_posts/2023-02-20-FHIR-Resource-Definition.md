---
title: FHIR-Resource-Definition
comment: true
date: 2023-02-20 15:49:09
updated: 2023-07-21 18:12:00
tags:
---

# FHIR资源定义

> 本文主要记录FHIRv4.3.0的资源`Resource`内容是如何定义的。实际数据交换时，资源可以用不同的格式来表示: `XML`、`JSON`、`Turtle`和`UML`. 未来批量数据`Bulk Data Formats`的定义也将发布. 

### Resource Definition
FHIR资源指一个规格标准定义如何展示和健康相关的某个概念, 当前版本一共有146种资源，主要包括临床信息，人员与组织，财务，安全和术语。资源可以用几种不同的方法来定义:
- 一种是层级表格，以逻辑视图的方式展示内容, 简称逻辑表`Logical table`
- UML 统一模型语言
- 片假XML语法
- 片假JSON
- 片假Turtle

除了上述语法，还有其他格式可以使用，包括`W3C schema`, `Schematron`, `JSON Schema`和`StructureDefinition`.

#### Logical Table 逻辑表
逻辑表以树形结构来表示资源，并固定几个特定列， 如名称、标志、类型等。相对其它表示方式，用表格的形式显得更加直观和整洁。
|Column     |Content       |
| ---- | ---- |
|Name       |资源中元素的名字，如XML中manifest清单，JSON/RDF的属性名。有些名字结尾有[x]代表下面详细的介绍。同时名字前面还有一个图标标记类型。     |
|Flags       | 一组标记表示实现时该如何处理该元素。比如这个元素必须被支持或它是可以选择的。  |
|Card       |基。 该元素在资源中允许出现的次数的下限和上限。    |
|Type       |元素的类型。注意，元素的类型有两种含义，取决于该元素是否有定义的子元素。如果元素有子代，那么元素有一个匿名的类型，由子元素给定具体的类型。如果元素没有子代，那么该元素可以被预置的类型所指定。   |
|Description&Contraints       |元素的描述，约束的细节。比如那些情况适用编码的元素。    |


<center>表一. 逻辑表固定字段</center>

**举例:**
![](https://fastly.jsdelivr.net/gh/filess/img17@main/2023/02/20/1676860770692-3f9519a8-782f-40b4-b462-4a1523e51b2a.png)

**对于Type类型的图标：**
- <img src="http://hl7.org/fhir/icon_resource.png" style="display: inline; margin:auto;" alt="resource" />代表一个基础资源。
- <img src="http://hl7.org/fhir/icon_element.gif" style="display: inline; margin:auto;" alt="element"/>资源中的元素，同时此元素还可以定义子元素
- <img src="http://hl7.org/fhir/icon_choice.gif" style="display: inline; margin:auto;" alt="type choice"/>此元素可以有多个不同的类型
- <img src="http://hl7.org/fhir/icon_primitive.png" style="display: inline; margin:auto;" alt="基础类型" />代表基础类型元素，基础类型都以小写字母开头
- <img src="http://hl7.org/fhir/icon_datatype.gif" style="display: inline; margin:auto;" alt="复合类型" />元素的数据类型描述了其它的元素，称为复合类型，复合类型用大写字母开头
- <img src="http://hl7.org/fhir/icon_reference.png" style="display:inline; margin:auto;" alt="reference">子元素可以引用另一个资源
- <img src="http://hl7.org/fhir/icon_reuse.png" style="display:inline; margin:auto;" alt="reuse">此元素和同一个资源内另一个元素内容一样
- <img src="http://hl7.org/fhir/icon_slice.png" style="display:inline; margin:auto;" alt="切片集"/>引入切片集合, 具体见profile中定义.
- <img src="http://hl7.org/fhir/icon_extension_complex.png"  style="display:inline; margin:auto;" alt="complet extension" />复杂嵌套扩展
- <img src="http://hl7.org/fhir/icon_extension_simple.png" style="display:inline; margin:auto;" alt="simple extension" />简单扩展，只扩展一个值没有嵌套
- <img src="http://hl7.org/fhir/icon_modifier_extension_complex.png" style="display:inline; margin:auto;" atl="complex modifier" />复杂修改扩展
- <img src="http://hl7.org/fhir/icon_modifier_extension_simple.png" style="display:inline; margin:auto;" alt="simple modifier" /> 简单修改扩展
- <img src="http://hl7.org/fhir/icon_profile.png" style="display:inline; margin:auto;" alt="rootprofile" />逻辑概貌的根

**对于Flag标志的图标**
- <code>?!</code>: 这个元素是一个正在建设的元素
- <code>S</code>: 这个元素必须被支持
- <code>Σ</code>: 此元素是汇总集合的一部分
- <code>I</code>: 此元素定义了约束或被约束
- <code>《A》</code>抽象类型
- <code>《I》</code>此资源是接口定义
- <a style="padding-left: 3px; padding-right: 3px; border: 1px grey solid; font-weight: bold; color: black; background-color: #ffe6e6" >TU</a>此元素是试用状态
- <a style="padding-left: 3px; padding-right: 3px; border: 1px grey solid; font-weight: bold; color: black; background-color: #e6ffe6">N</a>该元素的标准状态为规范性，既正式状态。
- <a style="padding-left: 3px; padding-right: 3px; border: 1px grey solid; font-weight: bold; color: black; background-color: #efefef" >D</a>该元素的标准状态是起草阶段。

**其它注意项**
- 资源和元素是大小写敏感的
- 任何元素是基础类型的，它会有一个`value`的属性来表示具体的数值
- 元素都有一个基数，来表示这个元素会出现或者必须出现多少次
- 元素复用方面，如果子元素和另一个子元素有相同的数据类型，那么可以这样表示：`see[name]`括号中name是另子元素的名称。
- 每个元素名都会在数值字典里有正式的定义名，通过超链接找到对应的关系。
- 有些元素可能会有`id`的属性用来内部引用，上面的例子没有显示出来这个`id`。
- FHIR的元素永不为空。如果有一个元素在资源里出现，那么它要么必须有值，要么它的子元素定义，或者其他扩展。
- 基础类的元素是所有资源共有的，所以不会在表格中展示。这些共用的元素在基类[Resource](http://hl7.org/fhir/resource.html)和[DomainResource](http://hl7.org/fhir/domainresource.html)中定义。
元素的数据类型表示一般是type类型直接体现的，但是有两个特例：
- 如果元素支持多个类型（名字结尾[x]）, 那么类似可以是一系列选项，用`|`来分割。
- 如果一个类型是`Reference`或`canonical`, 那么数据类型将直接列出可能的引用或者profile链接。如果链接到profile, 参考的类型可能被`profiled`， 比如元素的实例必须遵循特定profile或者一组profile列表。特定的url使用`{}`来表示。

#### 数据类型的选择
有些元素可能有多个数据类型的选择，这样的情况需要使用`name[x]`来定义元素名，`[x]`指定实际使用的数据类型。如果想要某个元素只重复一次数据类型，那么它的基数只能是1. 

#### 格式的序列化
可以使用如下方式序列化资源：
- JSON
- XML
- RDF(Turtle)

系统必须在[Capability Statement](http://hl7.org/fhir/capabilitystatement.html)声明它支持的格式. 如果一个服务器收到它不支持的格式请求，那么需要返回`406 Not Acceptable`. 如果客户端post一个不支持的格式，那么返回`416 Unsupported Media Type`.
比较推荐的做法是支持JSON和XML格式，以适用于不同的技术栈。RDF比较适用于数据分析而不是数据交换。

#### 批量数据格式（建设中）
当FHIR需要交换批量数据如1000条以上数据的时候，批量数据格式就可以用上了。目前建议的支持格式有：
- ND-JSON(New line delimited JSON)
- Google Protobuf
- Apache Parquet/Avro

#### 外部链接
- [FHIR Resource Definition](http://hl7.org/fhir/formats.html#)
