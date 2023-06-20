---
title: Use CDS Authoring Tool
comment: true
date: 2023-06-12 15:26:30
tags:
---
<script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=ca-pub-2415109016615233"
     crossorigin="anonymous"></script>

# 使用CDS作者工具

> CDS Connect 作者工具是一种用于创建临床决策支持规则的工具。此工具提供了一个用户友好的界面，供医疗保健专业人员和技术人员使用。它允许用户创建、编辑和管理临床决策支持规则，这些规则可以帮助医生在临床实践中做出更明智的决策。

## 导言
`CDS Connect`项目开源了一个工具[Authoring tool](https://github.com/AHRQ-CDS/AHRQ-CDS-Connect-Authoring-Tool)用于快速创建临床辅助的规则、内容。有了这个工具可以方便地编写CQL（Clinical Quality Language）表达，而不用非常了解语法细节。该工具使用基于标准化的临床知识表示形式，如CQL和FHIR（Fast Healthcare Interoperability Resources）。用户可以根据特定的临床指南、研究结果和实践经验，使用这些规则来定义临床决策的逻辑和条件。CDS Connect 作者工具还提供了验证和测试功能，以确保创建的规则能够正确地应用于真实的临床环境。它还支持与其他系统的集成，可以将创建的规则导出为可用于不同临床决策支持系统的格式。下面就配合官方文档，简单地用一个例子作一下演示。

### 基本概念
- `Element`元素。CDS每条规则由元素组成，每个元素描述了一条规则用于决定某个患者是否满足临床辅助推荐的标准。
- `Inclusion`包含。即患者满足某个元素的一条标准。
- `Exclusion`排除。即患者不满足某个推荐要求。
- `Subpopulation`亚群体。即亚群体需要更多的具体推荐才能作出决定。
- `Base Element`基础元素。可以自定义的基础元素。
- `Recommendation`推荐。当满足条件是cds给出的建议内容。
- `Parameter`参数。可以在运行时动态的输出参数。

### 创建和编辑元素流程
通常步骤包括：
1. 首先选择一个元素，元素通常是一个FHIR资源类型，如`Condition`或`Observation`。元素的类型也会取决哪些数据需要从患者数据（病例记录）中来。具体的元素类型有：
	- `Allergy Intolerance`过敏不耐受: Instances of the FHIR `AllergyIntolerance` resource type.
	- `Base Elements`基本元素: Re-usable elements defined in the "Base Elements" tab. 可重复使用的基本元素。
	- `Condition`病情: Instances of the FHIR `Condition` resource type.
	- `Demographics`人口统计学: Age or Gender as specified in an instance of the FHIR Patient resource type
	- `Device`设备: Instances of the FHIR `Device` resource type.
	- `Encounter`就诊: Instances of the FHIR `Encounter` resource type.
	- `External CQL`外部CQL: Named CQL definitions, parameters, and functions from CQL files uploaded in the "External CQL" tab. 来自“外部CQL”标签中上传的命名CQL定义、参数和函数.
	- `Immunization`免疫: Instances of the FHIR `Immunization` resource type
	- `Medication Statement`用药记录: Instances of the FHIR `MedicationStatement` resource type
	- `Medication Request`用药请求: Instances of the FHIR `MedicationRequest`(STU3/R4) or MedicationOrder(DSTU2) resource type
	- `Observation`观测: Instances of the `Observation` resource type
	- `Parameters`参数: Parameter values for parameters defined in the "Parameters" tab. 在“参数”标签中定义的参数值.
	- `Procedure`手术: Instances of the FHIR `Procedure` resource type
	- `ServiceRequest`服务请求: Instances of the FHIR `ServiceRequest` resource type, available only in FHIR R4

2. 然后根据所选元素类型，赋予更多的具体含义，如赋予最少一个键值对（这里的键值对是指一组code，这些code会用来匹配患者病历的记录。举个例子，糖尿病会包含多个code来代表不同糖尿病诊断，如一型、二型糖尿病。同时不同的编码系统如ICD-9,ICD-10,SNOMED-CT又有不同值。所以使用键值对可以方便地把所有对应的编码code一次性地和患者电子病历记录匹配起来）或者code编码（单独赋予code需要指定编码系统并验证，如果是第三方编码系统需要指定FHIR兼容的URL），如`Condition`->`Diabetes`症状对应糖尿病，`Observation`->`LDL Cholesterol Test`观测对应LDL胆固醇检测；又如通过表单提供额外的信息(目前仅用于人口统计学)。

3. 命名元素。所有的元素可以被自定义命名，以便更好地体现用意。

4. 备注元素。所有元素也支持被标注，添加评论, 但是评论是不会参与决策逻辑的判别。评论主要的应用场景有：
	- 提供创建该元素的理由，或指示在创建该元素时做出的决策以及为何做出该决策。
    - 提供对该元素所包含的复杂逻辑的简要总结，或进一步解释构建该元素的目的。
    - 指示元素逻辑的来源并提供必要的参考文献。
    - 指示实施者可能基于特定场地需求而希望修改的逻辑部分。

5. 修改结果。通过自定义修改或者选择内建的表达式进一步过滤结果，得到元素类型需要得出的结果。工具自带的修改器能覆盖一般的场景，同时修改器还可以串联逻辑，应用多个过滤条件得到最终结果。

5.1 （可选）使用自建修改器定义逻辑。如果作者想进行更加精准的控制，可以自建逻辑，有的时候自建修改器可能不如外部导入CQL方便。当前版本（June, 2023）自定义修改器只能对 FHIR 资源实例的列表进行过滤。未来的版本可能提供额外的功能，如排序和返回特定属性。

6.测试artifact。在将 CDS 逻辑部署到测试环境之前进行测试，可以让作者在流程早期发现和修复错误，从而节省时间和金钱。CDS创作工具的测试允许作者上传其自己合成的测试患者，这些患者以FHIR资源形式存在。现存的患者测试数据构造工具主要 [CQL Testing Framework](https://github.com/AHRQ-CDS/CQL-Testing-Framework) , [Synthea](https://github.com/synthetichealth/synthea) , [ClinFHIR](http://clinfhir.com/) , and [FHIR® Shorthand](https://build.fhir.org/ig/HL7/fhir-shorthand/) .然后，作者可以针对其中一个或多个患者运行他们的CDS逻辑，并检查结果以确定其是否符合预期。CDS创作工具目前不提供测试患者编辑器，也没有提供结果自动验证的机制（例如，“测试断言”）。对于更高级的测试功能，请考虑使用CDS Connect的[CQL测试框架](https://github.com/AHRQ-CDS/CQL-Testing-Framework)。


虽然元素本身可以很有用，但通常它们会与其他元素结合使用，以表示更复杂的思想或要求。CDS 作者工具支持以下元素组合方式：
- 与（And）：要求一组布尔元素中的每个元素都为真。
- 或（Or）：要求一组布尔元素中至少有一个元素为真。
- 缩进组（Indented Group）：将一组元素组合在一起，表示一个单一的逻辑单元。
- 交集（Intersect）：找出一组元素中每个元素中都出现的项目集合。使用“交集”组合方式表示应检查每个元素，并仅将与每个元素匹配的项目包含在交集结果中。例如，如果将表示已确认的心肌梗塞的元素与表示过去六个月内的心肌梗塞的元素使用“交集”进行组合，结果将仅包含同时在两个元素集合中的心肌梗塞（即已确认且在过去六个月内）。“交集”组合方式只能在“基本元素（Base Elements）”选项卡中使用“列表操作（List Operations）”元素类型进行应用。
- 并集（Union）：将多个元素中的项目合并为一个项目集合。使用“并集（Union）”组合方式表示应将所有元素中的所有项目合并为一个项目集合。例如，使用“并集”将LDL-c元素与HDL-c元素组合，将得到所有的LDL-c和HDL-c观测结果。并集只能在“基本元素（Base Elements）”选项卡中使用“列表操作（List Operations）”元素类型应用“并集”组合方式。

### Subpopulation亚群体页
作者可以使用“亚人群（Subpopulations）”选项卡来指定将患者分组，组内的亚群体适用标准为更具体的相关建议。虽然亚人群不是必需的，但对于需要提供更细致建议或与特定人群相关的建议而言，它们非常有用。例如，他汀类药物artifact可能会针对10年风险评分为8%的患者和10年风险评分为12%的患者提供不同强度的建议。“亚人群”选项卡允许作者创建所需的亚人群。对于每个亚人群，作者必须指定一个唯一的名称，然后根据上述部分的描述创建和组合元素。

### 基本元素
作者可以使用“基本元素（Base Elements）”创建artifact，基础元素可以是在上下文中多次使用。这样一来，常用元素只需定义一次，就可以在需要的任何地方使用，并可进行进一步修改。作者可以在任何特定上下文之外定义独立的元素，并将这些元素按原样导出为CQL。基于基本元素创建的元素会以浅蓝色进行阴影处理，以便更容易区分。此外，在元素定义的内容中使用“基本元素”标签列出了原始基本元素的名称。如果您点击基本元素名称最右侧的链接图标，将直接进入基本元素的定义页面。请注意，在使用基本元素时会有一些限制。其中之一是您不能删除它。要删除正在使用的基本元素，必须首先删除（或编辑）其所有用到的地方。另一个限制是，如果删除应用表达式会改变基本元素的返回类型，则无法删除该表达式。这确保修改基本元素不会使其使用变得无效。为了删除表达式，请首先删除工件中所有对基本元素的使用。此外，在正在使用的基本元素上无法添加修改器，除非修改器不会改变元素的整体返回类型。如果修改器会改变基本元素的返回类型，则必须先删除工件中所有对基本元素的使用，然后再添加修改器。

### 推荐页面Recommendation
推荐页面是CDS artifact在执行后会给临床医生一个推荐，推荐的具体内容可以被编辑并伴随合理的推荐理由。推荐也可以分亚群定制不同的内容。

### 参数
在CDS运行时输入参数可以动态的输出结果。参数可以有默认值，主要类型有:
`Boolean, Code, Concept, Integer, DateTime, Decimal, Quantity, String, Time, Interval<Integer>, Interval<DateTime>, Interval<Decimal>, and Interval<Quantity>`

### 错误处理
作者使用“处理错误”选项卡来定义在特定错误条件发生时向最终用户提供哪些信息。例如，作者可能希望在患者健康记录中缺少必要数据时返回错误消息。处理错误是可选的，对于某些用例可能不需要或不实用。

### 外部CQL
外部CQL可以通过页面上传并校验格式，外部CQL特别适合某些复杂数学计算和时间关系此工具处理不了的情况。同时外部CQL是read-only的类型。

