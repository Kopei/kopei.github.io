---
title: Use CDS Authoring Tool
comment: true
date: 2023-06-12 15:26:30
tags:
---
<script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=ca-pub-2415109016615233"
     crossorigin="anonymous"></script>

## 导言
`CDS Connect`项目开源了一个工具[Authoring tool](https://github.com/AHRQ-CDS/AHRQ-CDS-Connect-Authoring-Tool)用于快速创建临床辅助的规则、内容。有了这个工具可以方便地编写CQL（Clinical Quality Language）表达，而不用非常了解语法细节，下面就配合官方文档，简单地用一个例子作一下演示。

### 基本概念
- `Element`元素。CDS由元素组成，每个元素描述了一条标准用于决定某个患者是否满足临床辅助推荐的标准。
- `Inclusion`包含。即患者满足某个元素的一条标准。
- `Exclusion`排除。即患者不满足某个推荐要求。
- `Subpopulation`子群。即子群体需要更多的具体推荐才能作出决定。

### 创建和编辑元素流程
通常步骤包括：
1. 选择一个元素，元素通常是一个FHIR资源类型，如`Condition`或`Observation`。元素的类型也会觉得那些数据需要从患者数据中来。具体的元素类型有：
- `Allergy Intolerance`过敏不耐受: Instances of the FHIR `AllergyIntolerance` resource type
- `Base Elements`基本元素: Re-usable elements defined in the "Base Elements" tab. 可重复使用的基本元素。
- `Condition`病情: Instances of the FHIR `Condition` resource type
- `Demographics`人口统计学: Age or Gender as specified in an instance of the FHIR Patient resource type
- `Device`设备: Instances of the FHIR `Device` resource type
- `Encounter`就诊: Instances of the FHIR `Encounter` resource type
- `External CQL`外部CQL: Named CQL definitions, parameters, and functions from CQL files uploaded in the "External CQL" tab. 来自“外部CQL”标签中上传的命名CQL定义、参数和函数.
- `Immunization`免疫: Instances of the FHIR `Immunization` resource type
- `Medication Statement`用药记录: Instances of the FHIR `MedicationStatement` resource type
- `Medication Request`用药请求: Instances of the FHIR `MedicationRequest`(STU3/R4) or MedicationOrder(DSTU2) resource type
- `Observation`观测: Instances of the `Observation` resource type
- `Parameters`参数: Parameter values for parameters defined in the "Parameters" tab. 在“参数”标签中定义的参数值.
- `Procedure`手术: Instances of the FHIR `Procedure` resource type
- `ServiceRequest`服务请求: Instances of the FHIR `ServiceRequest` resource type, available only in FHIR R4
2. 根据所选元素类型，赋予更多的具体含义，如赋予最少一个键值对（这里的键值对是指一组code，这些code会用来匹配患者病历的记录。举个例子，糖尿病会包含多个code来代表不同糖尿病诊断，如一型、二型糖尿病。同时不同的编码系统如ICD-9,ICD-10,SNOMED-CT又有不同值。所以使用键值对可以方便地把所有对应的编码code一次性地和患者电子病历记录匹配起来）或者code编码，如`Condition`->`Diabetes`症状对应糖尿病，`Observation`->`LDL Cholesterol Test`观测对应LDL胆固醇检测；又如通过表单提供额外的信息。

3. 修改结果。通过自定义修改或者选择内建的表达式进一步过滤结果，得到元素类型需要得出的结果。




