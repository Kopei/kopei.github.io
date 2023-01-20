---
layout: post
title: Eyes on FHIR Implement Guide（2)
categories: [standard]
description: 
keywords: ophathalmology, HL7, FHIR
catalog: true
multilingual: false
tags: FHIR, Standard,IG
date: 2022-12-24
comment: true
---
# FHIR眼科实施指南IG（2）
> 2021年8月，FHIR patient care小组发布了FHIR眼科实施指南（IG）0.1.0版本，此版本只针对眼底相关疾病，并且通过了FHIR connectathon测试，可以双向与真实世界的EMR、诊断设备和PACS通信。

## IG主要内容
实施指南(Implement Guide)的主要内容可分为：
- 患者旅程
- 概貌（Profiles）
- 15个真实用例
- 指引规范
- 术语
- 贡献者
- 产出物

## 概貌（Profiles）
HL7中国的wiki把`profile`翻译成`概貌`，也有翻译成`配置或规范`, 因为`profiling`翻译成`概貌化`似乎没有`规范化`更加准确。0.1版本中有如下临床概貌已经被定义：
### 眼科观测（Observations）
- **基础眼科观察概貌**，此`ObservationBase`概貌描述仅限眼科的观察，下表1是必须实现的字段。

|Name     |Cardinality       |Type       |
| --- | --- | --- |
|status       |1..1       |[code](http://build.fhir.org/ig/HL7/fhir-eyecare-ig/ValueSet-observation-final-status.html)       |
|category       |0..*       |[CodeableConcept](http://hl7.org/fhir/R4/codesystem-observation-category.html#4.3.14.232.2)       |
|code       |1..1       |[CodeableConcept](http://hl7.org/fhir/R4/valueset-observation-codes.html)       |
|subject       |1..1       |[Reference(Patient)](http://hl7.org/fhir/R4/patient.html)       |
|bodySite       |0..1       |[CodeableConcept](http://hl7.org/fhir/R4/valueset-body-site.html#expansion)       |
|bodySite.extension       |0..*       |[Reference(BodyStructure)](http://hl7.org/fhir/R4/bodystructure.html)       |
|bodySite.extension.value       |1..1       |Reference(BodyStructure\|[Ocular anatomical location](http://build.fhir.org/ig/HL7/fhir-eyecare-ig/StructureDefinition-body-structure-eye.html))       |

<center>表一. 基础眼科观察profile必须实现的字段</center>

- **眼部解剖位置概貌**。此`BodyStructureEye`概貌将眼部分为眼球、眼周和眼眶。由于眼部的位置需要更加细粒度的描述，所以需要使用`BodyStructure.locationQualifier`和`BodyStructure.location`绑定来精确地描述眼部解剖位置。

|Name     |Cardinality       |Type       |
| --- | --- | --- |
|location       |0..1       |[CodeableConcept](http://build.fhir.org/ig/HL7/fhir-eyecare-ig/ValueSet-body-site-eye.html#root)       |
|locationQualifier       |0..*       |[CodeableConcept](http://build.fhir.org/ig/HL7/fhir-eyecare-ig/ValueSet-qualifiers.html)       |

<center>表二. 眼部解剖位置概貌profile必须实现的字段</center>

#### 临床观察（测量类的发现）
- **眼压IOP**`ObservationIOP`，患者眼内(眼球内)压力的测量值(单位：mmHg)。

|Name     |Cardinality       |Type       |
| --- | --- | --- |
|  status     |   1..1    |code(final or amended)       |
| code      |1..1       |CodeableConcept    |
|subject       |1..1       |Reference(Patient)       |
|  value[x]*     |1..1       |Quantity       |
|value[x].value       | 1..1      |decimal       |
|value[x].unit       |1..1       |string       |
|value[x].system      |1..1       |uri       |
|value[x].code       |1..1     |code       |
|bodySite       |0..1       |CodeConcept       |
|bodySite.extension      |0..*       |Extension(bodySite)      |
|bodySite.extension.value[x]     |1..1      |Reference(BodyStructure\|Ocular anatomical location)       |
| method      |0..1       | [codeableConcept](http://hl7.org/fhir/R4/valueset-observation-methods.html)     |
|method.coding       |1..*       |[Coding](http://build.fhir.org/ig/HL7/fhir-eyecare-ig/ValueSet-iop-methods.html)       |
<center>表三.眼压测量概貌profile</center>
*[x]代表元素可以有多个类型

- **视力VA**`ObservationVisualAcuity`, 可测量的检测还包括视力`Visual Acuity`概貌。

|Name     |Cardinality       |Type       |
| --- | --- | --- |
|  status     |   1..1    |code(final or amended)       |
| category  | 0..* |         CodeableConcept                    |
| code      |1..1       |CodeableConcept    |
|subject       |1..1       |Reference(Patient)       |
|bodySite       |0..1       |CodeConcept       |
|bodySite.extension      |0..*       |Extension(bodySite)      |
|bodySite.extension.value[x]     |1..1      |Reference(BodyStructure\|Ocular anatomical location)       |
| method      |1..1       | [codeableConcept](http://hl7.org/fhir/R4/valueset-observation-methods.html)     |
<center>表四.视力测量概貌</center>

#### 临床观察（观测类的发现）
其它观测类的临床发现用这个`ObservationEyeRegionFinding`概貌描述，此概貌也可以用`Condition`资源，可以来描述其它非眼科类的观察，比如：
- 很可能与疾病无关的观察，如患者进入诊室的步态。
- 可能相关的观察，如甲状腺肿大，和甲状腺相关的眼科疾病。
- 高度相关的观察，如高血压。高血压是潜在非眼部或系统性的致盲原因，例如严重的高血压性视网膜病变或视网膜血管阻塞。

### 眼科诊断/疾病
`ConditionBase`这个概貌用来描述过去或者当前的某个眼科疾病诊断。虽然这个概貌和上述`临床其他观测类发现`是参考了同一个概貌(`Condition`资源和眼科键值对值集组合), 但是此处所用应为实际临床诊断，而上述仅仅为临床其他类观测。并且眼部解剖位置概貌`BodyStructureEye`概貌在此处需组合使用。

|Name     |Cardinality       |Type       |
| --- | --- | --- |
|  Condition.code     |   1..1    |CodeableConcept(Ophthalmology Condition ICD10 and SNOMED codes ValueSet) |
| Condition.bodySite  | 0..* |         CodeableConcept(anatomical location)                    |
| Condition.bodySite.extension      |0..*       |Extension    |
| Condition.bodySite.extension.value[x]      |1..1     |[Reference](BodyStructure\|http://hl7.org/fhir/uv/eyecare/StructureDefinition/body-structure-eye)    |
|Condition.subject|1..1|Reference(Patient\|Group)|
<center>表五.眼科诊断概貌</center>

### 眼科干预程序
`ProcedureBase`眼科基本的干预程序，搭配眼部解剖位置概貌组合使用。

|Name     |Cardinality       |Type       |
| --- | --- | --- |
|  Procedure.code     |   1..1    |CodeableConcept([OphthalmologyProceduresValueSet](http://build.fhir.org/ig/HL7/fhir-eyecare-ig/ValueSet-procedures.html)) |
| Procedure.bodySite  | 0..* |         CodeableConcept(anatomical location)                    |
| Condition.bodySite.extension      |0..*       |Extension    |
| Condition.bodySite.extension.value[x]      |1..1     | [Reference](BodyStructure\|http://hl7.org/fhir/uv/eyecare/StructureDefinition/body-structure-eye)    |
<center>表六.眼科干预程序概貌</center>

### 诊断检查报告
`OphthalDiagnosticReport`定义了眼科诊断报告概貌。

|Name     |Cardinality       |Type       |
| --- | --- | --- |
|  DiagnosticReport.category     |   1..*   |CodeableConcept(service category) |
| DiagnosticReport.category.coding             |   1..1      |  Coding (ophthalCode)                             |
| DiagnosticReport.category.coding.system   |   1..1     |  uri(固定值：http://snomed.info/sct)          |
| DiagnosticReport.category.coding.code             |   1..1      |  code(固定值：394594003)          |
<center>表七.眼科诊断报告概貌</center>

#### 视野检查
`ObservationVisualField`视野检查概貌，此概貌可以单独用在视野检查观察。

|Name     |Cardinality       |Type       |
| --- | --- | --- |
|  status     |   1..1    |code(final or amended)       |
|    category |0..* | CodeableConcept(ObservationCategoryCodes) |
| code      |1..1       |CodeableConcept(LOINCCodes)    |
|subject       |1..1       |Reference(Patient)       |
|bodySite       |0..1       |CodeableConcept       |
|bodySite.extension      |0..*       |Extension(bodySite)      |
|bodySite.extension.value[x]     |1..1      |Reference(BodyStructure\|Ocular anatomical location)       |
<center>表八.视野检查概貌</center>

#### 视野诊断报告
`OphthalDiagnosticReportForVisualField`视野检查报告概貌。


|Name     |Cardinality       |Type       |
| --- | --- | --- |
|  DiagnosticReport.category     |   1..*   |CodeableConcept(service category) |
| DiagnosticReport.code.coding   |   1..1     |  Coding(vfCode)          |
| DiagnosticReport.code.coding.system     |   1..1      |  uri(固定值：http://snomed.info/sct)          |
| DiagnosticReport.code.coding.code             |   1..1      |  code(固定值：103752008)          |
| DiagnosticReport.result|   0..*      |  Reference(Observation \| VF Observations)      |
<center>表九.视野检查报告概貌</center>

#### OCT黄斑检查
`ObservationOCTMacula`, OCT黄斑检查概貌，一般和诊断报告一起使用。

|Name     |Cardinality       |Type       |
| --- | --- | --- |
|  status     |   1..1    |code(final or amended)       |
|    category |0..* | CodeableConcept(ObservationCategoryCodes) |
| code      |1..1       |CodeableConcept(LOINCCodes)    |
|subject       |1..1       |Reference(Patient)       |
|bodySite       |0..1       |CodeableConcept       |
|bodySite.extension      |0..*       |Extension(bodySite)      |
|bodySite.extension.value[x]     |1..1      |Reference(BodyStructure\|Ocular anatomical location)       |
<center>表十.OCT黄斑检查概貌</center>

#### OCT黄斑诊断报告
`OphthalDiagnosticReportOCTMacula`, OCT黄斑诊断报告。
|Name     |Cardinality       |Type       |
| --- | --- | --- |
|  DiagnosticReport.category     |   1..*   |CodeableConcept(service category) |
| DiagnosticReport.code.coding   |   1..1     |  Coding(maculaCode)          |
| DiagnosticReport.code.coding.system     |   1..1      |  uri(固定值：http://loinc.org)          |
| DiagnosticReport.code.coding.code             |   1..1      |  code(固定值：57119-0)          |
| DiagnosticReport.result|   0..*      |  [Reference](Observation\|http://hl7.org/fhir/uv/eyecare/StructureDefinition/observation-oct-macula) |
<center>表十一.黄斑诊断报告概貌</center>

#### OCT RNFL视网膜神经纤维层检查
`ObservationOCTRNFL`, OCT视网膜神经纤维层检查概貌，一般和诊断报告一起使用。

|Name     |Cardinality       |Type       |
| --- | --- | --- |
|  status     |   1..1    |code(final or amended)       |
|  category |0..* | CodeableConcept(ObservationCategoryCodes) |
| code      |1..1       |CodeableConcept(LOINCCodes)    |
|subject       |1..1       |Reference(Patient)       |
|bodySite       |0..1       |CodeableConcept       |
|bodySite.extension      |0..*       |Extension(bodySite)      |
|bodySite.extension.value[x]     |1..1      |Reference(BodyStructure\|Ocular anatomical location)       |
<center>表十二.OCT RNFL检查概貌</center>

#### OCT黄斑诊断报告
`OphthalDiagnosticReportOCTRNFL`, OCT视网膜神经纤维层诊断报告。
|Name     |Cardinality       |Type       |
| --- | --- | --- |
|  DiagnosticReport.category     |   1..*   |CodeableConcept(service category) |
| DiagnosticReport.code.coding   |   1..1     |  Coding(rnflCode)          |
| DiagnosticReport.code.coding.system     |   1..1      |  uri(固定值：http://loinc.org)          |
| DiagnosticReport.code.coding.code             |   1..1      |  code(固定值：86291-2)          |
| DiagnosticReport.result|   0..*      |  [Reference](Observation | OCT RFNL Observations) |
<center>表十三.OCT RNFL诊断报告概貌</center>

#### 外部链接
- [Eyes on FHIR Profiles](http://build.fhir.org/ig/HL7/fhir-eyecare-ig/profiles.html)

