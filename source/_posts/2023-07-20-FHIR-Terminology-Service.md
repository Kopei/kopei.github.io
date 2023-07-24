---
title: FHIR Terminology Service
comment: true
date: 2023-07-20 11:12:12
tags: FHIR
---
# FHIR Terminology Service 术语服务

> 本文将正对`HL7 FHIR R4`的术语模块做一下介绍，包括官方文档和实现指引。

### FHIR 术语模块的设计思路

HL7 的术语模块提供了一个整体的设计用于指引大家如何使用术语资源，以及相应的操作，数据类型定义，外部和自定义术语库如何展示和交换。最终的目的是为了提供标准的术语服务支持 FHIR 资源间使用标准的编码系统。
术语模块主要有 5 个资源：

- CodeSystem: 编码系统，描述术语的关键元素和定义。
- ConceptMap: 定义一个编码系统的概念与另一个的映射关系。
- ValueSet: 将一个或多个编码系统中用于特定目的的一组代码进行组合, 形成有特定含义的值集。
- NamingSystem: 命名系统。
- TerminologyCapabilities: 术语能力声明。

他们之间的关系可以用下图表示，
![relation](/images/screenshots/terminology-module-relationships.png)

实际使用中，我们在对接不同系统时可能只需做一下转换，把一个编码系统的值集映射成另一个编码系统的值集, 可能不一定需要命名系统，如下图所示。
![vsmapping](/images/screenshots/valuesets-mapping.png)

### 术语服务

HL7 也给出标准的术语服务实施规范，用户在使用这个服务的时候可以做到不用十分了解编码系统、值集和映射概念。如果一个服务能满足如下列表要求，就可以在 FHIR 能力声明中宣称符合 FHIR 术语服务标准[Terminology Service Capability Statement](https://fhir-ru.github.io/capabilitystatement-terminology-server.html)：

- 安全
- 值集拓展, $expand 从值集中扩展返回一组编码列表，实际使用时类似高级 search。
- 概念查找/分解, $lookup 查找资源的一个概念，值集或关系。
- 值集验证， $validate-code 验证一个编码在值集中是否有效。
- 包容性测试，$subsumes 查询两个代码之间是否存在（如果有的话）包含关系。
- 批量验证
- 翻译,$translate 使用 ConceptMap 资源，查找特定源概念被映射到的目标编码系统的概念。
- 批量翻译
- 可以维护一个闭环表

#### 服务的安全要求

SSL 加密传输是强制要求。如果值集系统允许被实时维护，认证和审计机制也是必要的。

#### 熟悉基本概念

由于术语服务是基于几个基本资源的，所以熟悉这些资源和使用方法对实现十分重要。基本资源包括：`CodeSystem`,`ValueSet`, `ConceptMap`. 以及熟悉如何在 FHIR 中使用编码。

#### 如何在 FHIR 中使用编码

FHIR 中的资源很多会用到编码，这些编码往往是一串固定的值，代表一个特定的含义或概念。FHIR 这里的编码定义都是通过一对组合:`system`和`code`. `system`的值是一个 url 指出哪里定义了这个编码，这个值需要区分大小写。
|Key|Value|
|---|---|
|system|URI 指定编码的位置|
|version|版本|
|code|字符串表示一个概念|
|display|描述文字|

编码在资源中定义时可以有多种数据类型，但都代表编码。

<table>
 <tbody><tr><td colspan="2">当编码在资源中定义时，可以有如下数据类型</td></tr>
 <tr><td>code</td><td>这数据类型只展示编码`code`, `system`隐含在元素中。</td></tr>
 <tr><td>Coding</td><td>这个数据类型定义一个标准的`code`和`system`对。</td></tr>
 <tr><td>CodeableConcept</td><td>`coding`另加上一个直白的文本字段.</td></tr>
 <tr><td colspan="2">另外，非资源元素定义时，如下字段也可以带着编码.或者内容当成编码，并绑定一个值集。</td></tr>
 <tr><td>Quantity</td><td>数量这个字段可以用`system`and<i>code</i>定义单位的类型.</td></tr>
 <tr><td>string</td><td>有的时候string字符串也可以用来控制某个元素固定的几个值.</td></tr>
 <tr><td>uri</td><td>类似string, uri也可以当成编码元素</td></tr></tbody>
</table>

#### 选择编码系统

`system`对应的 url 需要指向一个编码系统。编码系统可以有如下地址可以引用:

- 本规范指定的编码系统仓库[code system registry](https://hl7.org/fhir/R4/terminologies-systems.html)
- 编码系统发布者定义的 URI 或者 OID
- FHIR 社区的编码系统仓库，并且状态是`active`
- 在[HL7 OID registry](http://www.hl7.org/oid/index.cfm)注册的 OID

下表是所有外部编码系统：
| URI | Source | Comment | OID (for non-FHIR systems) |
| --- | --- | --- | --- |
| http://snomed.info/sct | SNOMED CT (IHTSDO) | See [Using SNOMED CT with FHIR](snomedct.html) | 2.16.840.1.113883.6.96 |
| http://www.nlm.nih.gov/research/umls/rxnorm | RxNorm (US NLM) | See [Using RxNorm with FHIR](rxnorm.html) | 2.16.840.1.113883.6.88 |
| http://loinc.org | LOINC (LOINC.org) | See [Using LOINC with FHIR](loinc.html) | 2.16.840.1.113883.6.1 |
| http://unitsofmeasure.org | UCUM: (UnitsOfMeasure.org) Case Sensitive Codes | See [Using UCUM with FHIR](ucum.html) | 2.16.840.1.113883.6.8 |
| http://ncimeta.nci.nih.gov | [NCI Metathesaurus](http://ncimeta.nci.nih.gov) | See [Using NCI Metathesaurus with FHIR](ncimeta.html) | 2.16.840.1.113883.3.26.1.2 |
| http://www.ama-assn.org/go/cpt | [AMA CPT codes](http://www.ama-assn.org/go/cpt) | See [Using CPT with FHIR](cpt.html) | 2.16.840.1.113883.6.12 |
| http://hl7.org/fhir/ndfrt | [NDF-RT (National Drug File – Reference Terminology)](http://www.nlm.nih.gov/research/umls/sourcereleasedocs/current/NDFRT/) | See [Using NDF-RT with FHIR](ndfrt.html) | 2.16.840.1.113883.6.209 |
| http://fdasis.nlm.nih.gov | [Unique Ingredient Identifier (UNII)](http://www.fda.gov/Drugs/InformationOnDrugs/ucm142438.htm) | See [Using UNII with FHIR](unii.html) | 2.16.840.1.113883.4.9 |
| http://hl7.org/fhir/sid/ndc | [NDC/NHRIC Codes](http://www.fda.gov/Drugs/InformationOnDrugs/ucm142438.htm) | See [Using NDC with FHIR](ndc.html) | 2.16.840.1.113883.6.69 |
| http://hl7.org/fhir/sid/cvx | [CVX (Vaccine Administered)](http://www2a.cdc.gov/vaccines/iis/iisstandards/vaccines.asp?rpt=cvx) | See [Using CVX with FHIR](cvx.html) | 2.16.840.1.113883.12.292 |
| urn:iso:std:iso:3166 | [ISO Country & Regional Codes](http://www.iso.org/iso/country_codes.htm) | See [Using ISO 3166 Codes with FHIR](iso3166.html) | 1.0.3166.1.2.2 |
| http://hl7.org/fhir/sid/dsm5 | [DSM-5](https://en.wikipedia.org/wiki/DSM-5) | Diagnostic and Statistical Manual of Mental Disorders, Fifth Edition (DSM-5) | 2.16.840.1.113883.6.344 |
| http://www.nubc.org/patient-discharge | [NUBC code system for Patient Discharge Status](http://www.nubc.org) | National Uniform Billing Committee, manual UB-04, UB form locator 17 | 2.16.840.1.113883.6.301.5 |
| http://www.radlex.org | [RadLex](http://www.radlex.org) | (Includes [play book](http://playbook.radlex.org) codes) | 2.16.840.1.113883.6.256 |
| ICD-9, ICD-10 | [WHO](http://www.who.int/classifications/icd/en/) & National Variants | See [Using ICD-[x] with FHIR](icd.html) | See ICD page for details |
| http://hl7.org/fhir/sid/icpc-1 | [ICPC (International Classification of Primary Care)](http://www.ph3c.org/) | [NHG Table 24 ICPC-1 (NL)](https://referentiemodel.nhg.org/tabellen/nhg-tabel-24-icpc1) | 2.16.840.1.113883.2.4.4.31.1 |
| http://hl7.org/fhir/sid/icf-nl | [ICF (International Classification of Functioning, Disability and Health)](http://www.who.int/classifications/icf/en/) | | 2.16.840.1.113883.6.254 |
| http://terminology.hl7.org/CodeSystem/v2-[X](/v) | [Version 2 tables](terminologies-v2.html) | [X] is the 4 digit identifier for a table; e.g. http://terminology.hl7.org/CodeSystem/v2-0203 | 2.16.840.1.113883.12.[X] |
| http://terminology.hl7.org/CodeSystem/v3-[X] | [A HL7 v3 code system](terminologies-v3.html) | [X] is the code system name; e.g. http://terminology.hl7.org/CodeSystem/v3-GenderStatus | see [v3 list](terminologies-v3.html) |
| https://www.gs1.org/gtin | [GTIN (GS1)](https://www.gs1.org) | Note: GTINs may be used in both [Codes](datatypes.html#Coding) and [Identifiers](datatypes.html#Identifier) | 1.3.160 |
| http://www.whocc.no/atc | [Anatomical Therapeutic Chemical Classification System (WHO)](http://www.whocc.no/atc/structure_and_principles/) | | 2.16.840.1.113883.6.73 |
| urn:ietf:bcp:47 | IETF language (see [Tags for Identifying Languages - BCP 47](http://tools.ietf.org/html/bcp47)) | This is used for identifying language throughout FHIR. Note that usually these codes are in a `code` and the system is assumed | |
| urn:ietf:bcp:13 | Mime Types (see [Multipurpose Internet Mail Extensions (MIME) Part Four - BCP 13](http://tools.ietf.org/html/bcp13)) | This is used for identifying the mime type system throughout FHIR. Note that these codes are in a `code` (e.g. [Attachment.contentType](datatypes.html#Attachment)) and in these elements the system is assumed. This system is defined for when constructing value sets of mime type codes | |
| urn:iso:std:iso:11073:10101 | Medical Device Codes (ISO 11073-10101) | See [Using MDC Codes with FHIR](mdc.html) | 2.16.840.1.113883.6.24 |
| http://dicom.nema.org/resources/ontology/DCM | DICOM Code Definitions | The meanings of codes defined in DICOM, either explicitly or by reference to another part of DICOM or an external reference document or standard | 1.2.840.10008.2.16.4 |
| http://hl7.org/fhir/NamingSystem/ca-hc-din | [Health Canada Drug Identification Number](http://www.hc-sc.gc.ca/dhp-mps/prodpharma/activit/fs-fi/dinfs_fd-eng.php) | A computer-generated eight-digit number assigned by Health Canada to a drug product prior to being marketed in Canada. [Canada Health Drug Product Database](http://www.hc-sc.gc.ca/dhp-mps/prodpharma/databasdon/index-eng.php) contains product specific information on drugs approved for use in Canada. | 2.16.840.1.113883.5.1105 |
| http://hl7.org/fhir/sid/ca-hc-npn | [Health Canada Natural Product Number](https://www.canada.ca/en/health-canada/services/drugs-health-products/natural-non-prescription/applications-submissions/product-licensing/licensed-natural-health-products-database.html) | A computer-generated number assigned by Health Canada to a natural health product prior to being marketed in Canada. | 2.16.840.1.113883.5.1105 |
| http://nucc.org/provider-taxonomy | [NUCC Provider Taxonomy](http://www.nucc.org/index.php/code-sets-mainmenu-41/provider-taxonomy-mainmenu-40/csv-mainmenu-57) | The Health Care Provider Taxonomy code is a unique alphanumeric code, ten characters in length. The code set is structured into three distinct "Levels" including Provider Type, Classification, and Area of Specialization. | 2.16.840.1.113883.6.101 |
| | Code Systems for Genetics | | |
| http://www.genenames.org | [HGNC: Human Gene Nomenclature Committee](http://www.genenames.org) | | 2.16.840.1.113883.6.281 |
| http://www.ensembl.org | [ENSEMBL reference sequence identifiers](http://www.ensembl.org) | Maintained jointly by the European Bioinformatics Institute and Welcome Trust Sanger Institute | not assigned yet |
| http://www.ncbi.nlm.nih.gov/refseq | [RefSeq: National Center for Biotechnology Information (NCBI) Reference Sequences](https://www.ncbi.nlm.nih.gov/refseq) | | 2.16.840.1.113883.6.280 |
| http://www.ncbi.nlm.nih.gov/clinvar | [ClinVar Variant ID](http://www.ncbi.nlm.nih.gov/clinvar) | NCBI central repository for curating pathogenicity of potentially clinically relevant variants | not assigned yet |
| http://sequenceontology.org | [Sequence Ontology](http://sequenceontology.org) | | not assigned yet |
| http://varnomen.hgvs.org | [HGVS: Human Genome Variation Society](http://varnomen.hgvs.org) | | 2.16.840.1.113883.6.282 |
| http://www.ncbi.nlm.nih.gov/projects/SNP | [DBSNP: Single Nucleotide Polymorphism database](http://www.ncbi.nlm.nih.gov/projects/SNP) | | 2.16.840.1.113883.6.284 |
| http://cancer.sanger.ac.uk/cancergenome/projects/cosmic | [COSMIC: Catalogue Of Somatic Mutations In Cancer](http://cancer.sanger.ac.uk/cancergenome/projects/cosmic) | | 2.16.840.1.113883.3.912 |
| http://www.lrg-sequence.org | [LRG: Locus Reference Genomic Sequences](http://www.lrg-sequence.org) | | 2.16.840.1.113883.6.283 |
| http://www.omim.org | [OMIM: Online Mendelian Inheritance in Man](http://www.omim.org) | | 2.16.840.1.113883.6.174 |
| http://www.ncbi.nlm.nih.gov/pubmed | [PubMed](http://www.ncbi.nlm.nih.gov/pubmed) | | 2.16.840.1.113883.13.191 |
| http://www.pharmgkb.org | [PHARMGKB: Pharmacogenomic Knowledge Base](http://www.pharmgkb.org) | PharmGKB Accession ID | 2.16.840.1.113883.3.913 |
| http://clinicaltrials.gov | [ClinicalTrials.gov](http://clinicaltrials.gov) | | 2.16.840.1.113883.3.1077 |
| http://www.ebi.ac.uk/ipd/imgt/hla | [European Bioinformatics Institute](http://www.ebi.ac.uk/ipd/imgt/hla) | | 2.16.840.1.113883.6.341 |

#### 编码系统的复杂表达

由于真实世界经常涉及通过组合多个概念或属性来创建复合术语，同时很多编码系统内部的概念之间会有各种关系，比如相等，包含，属于等等多种关系。编码系统如`SNOMED-CT`就设计了表达式来支持更细节的临床意义表达。这些特征可以在`CodeSystem`资源来展示，并通过上述的几种数据类型进行数据交换。

```javascript
{
  "system" : "http://snomed.info/sct",
  "code" : "80146002 |appendectomy| : 260870009 |priority| = 25876001 |emergency|"
}
## 阑尾炎切除手术紧急 concept: refinement(key=value)
```

#### 元素编码值绑定

当一个元素和值集绑定时，绑定有如下属性：
|Name|Description|
|---|---|
|Strength|绑定的强度，灵活度|
|Reference|URL 指定值集路径|
|Description|编码使用的描述信息|

几乎所有的元素都有被编码的数据类型，这个数据类型和一个值集绑定。绑定的强度可以有多种程度：

| strength   | Description                                                              |
| ---------- | ------------------------------------------------------------------------ |
| required   | 必须遵循，元素中的这个概念必须被指定值集                                 |
| extensible | 必须遵循，值集的绑定可以通过扩展方式提供可替代的概念。但不是扩展值集本身 |
| preferred  | 推荐使用特定编码来提高互操作性，但不是强制要求                           |
| example    | 没有例子可以使用来绑定值集                                               |

#### $expand 操作

术语服务的“值集扩展”是指将值集解析为包含在该值集中的个别代码的过程, 使其更容易在特定上下文中处理和解释代码。该过程确保了值集所代表的信息被明确理解，并且可以轻松地被各种医疗保健应用程序和系统使用。

```
### 扩展查询第23值集，过滤“abdo”

GET [base]/ValueSet/23/$expand?filter=abdo
Expanding a value set that is specified by the client (using JSON):

POST [base]/ValueSet/$expand
[other headers]

{
  "resourceType" : "Parameters",
  "parameter" : [
     {
     "name" : "valueSet",
     "resource" : {
       "resourceType" : "ValueSet",
     [value set details]
     }
   }
  ]
}
```

```
## 返回
HTTP/1.1 200 OK
[other headers]

<ValueSet xmlns="http://hl7.org/fhir">
  <!-- the server SHOULD populate the id with a newly created UUID
    so clients can easily track a particular expansion  -->
  <id value="43770626-f685-4ba8-8d66-fb63e674c467"/>
  <!-- no need for meta, though it is allowed for security labels, profiles -->

  <!-- other value set details -->
  <expansion>
    <!-- when expanded -->
    <timestamp value="20141203T08:50:00+11:00"/>
  <contains>
    <!-- expansion contents -->
  </contains>
  </expansion>
</ValueSet>
```

#### $lookup 概念查找

一个外部系统可以向术语服务器发出查询操作来获取关于特定的系统/代码组合的一组信息。服务器会返回用于显示和处理的信息。
例如，通过已知 valueset 23 其验证一个 codesystem 概念:

```
GET [base]/ValueSet/23/$validate-code?system=http://loinc.org&code=1963-8&display=test
```

通过指定一个 value set 来验证 CodeableConcept:

```
POST [base]/ValueSet/$validate-code
[other headers]

{
  "ResourceType" : "Parameters",
  "parameter" : [
    {
    "name" : "coding",
    "valueCodeableConcept" : {
      "coding" : {
        "system" : "http://loinc.org",
          "code" : "1963-8",
      "display" : "test"
      }
    }
  },
  {
    "name" : "valueSet",
    "resource": {
      "resourceType" : "ValueSet",
    [etc.]
    }
  }
  ]
}

## Response
HTTP/1.1 200 OK
[other headers]

{
  "resourceType" : "Parameters",
  "parameter" : [
    {
    "name" : "result",
    "valueBoolean" : false
  },
  {
    "name" : "message",
    "valueString" : "The display \"test\" is incorrect"
  },
  {
    "name" : "display",
    "valueString" : "Bicarbonate [Moles/volume] in Serum"
  }
  ]
}
```

#### $translate 概念转化

客户端可以向服务器请求将一个概念从一个值集转换为另一个值集。通常，这用于在编码系统之间进行转换（例如从 LOINC 转换为 SNOMED CT，或从 HL7 V3 代码转换为 HL7 V2 代码）。
例子：把 FHIR 复合状态映射成 v3

```
GET [base]/ConceptMap/$translate?system=http://hl7.org/fhir/composition-status
  &code=preliminary&source=http://hl7.org/fhir/ValueSet/composition-status
  &target=http://terminology.hl7.org/ValueSet/v3-ActStatus

HTTP/1.1 200 OK
[other headers]

{
  "resourceType" : "Parameters",
  "parameter" : [
    {
    "name" : "result",
    "valueBoolean" : true
    },
    {
      "name" : "outcome",
      "valueCoding" : {
        "system" : "http://terminology.hl7.org/CodeSystem/v3-ActStatus",
        "code" : "active",
      }
    }
  ]
}
```

### 推荐阅读

- [FHIR Terminology Server](https://tx.fhir.org/r4/)
- [Use codes in Resources](https://hl7.org/fhir/R4/terminologies.html)
- [SNOMED Expression](https://confluence.ihtsdotools.org/display/DOCSTART/7.+SNOMED+CT+Expressions)
