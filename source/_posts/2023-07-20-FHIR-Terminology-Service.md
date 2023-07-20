---
title: FHIR-Terminology-Service
comment: true
date: 2023-07-20 11:12:12
tags:
---
> 本文将正对`HL7 FHIR R4`的术语模块做一下介绍，包括官方指引和实现。

## FHIR术语模块的设计思路
HL7的术语模块提供了一个整体的设计用于指引大家如何使用术语资源，以及相应的操作，数据类型定义，外部和自定义术语库如何展示和交换。最终的目的是为了提供标准的术语服务支持FHIR资源间使用标准的编码系统。
术语模块主要有5个资源：
- CodeSystem
- ConceptMap
- ValueSet
- NamingSystem
- TerminologyCapabilities
他们之间的关系可以用下图表示，
![relation](/images/screenshots/terminology-module-relationships.png)

实际使用中，我们在对接不同系统时可能只需做一下转换，把一个编码系统的值集映射成另一个编码系统的值集, 可能不一定需要命名系统，如下图所示。
![vsmapping](/images/screenshots/valuesets-mapping.png)

### 术语服务
HL7也给出标准的术语服务实施规范，用户在使用这个服务的使用希望可以做到不用十分了解细节的编码系统、值集和映射概念。如果一个服务能满足如下列表要求，就可以在FHIR能力声明中宣称符合FHIR术语服务标准`[Terminology Service Capability Statement](https://fhir-ru.github.io/capabilitystatement-terminology-server.html)`.
- 安全
- 基本概念
- 值集拓展
- 概念查找/分解
- 值集验证
- 包容性测试
- 批量验证
- 翻译
- 批量翻译
- 可以维护一个闭环表

#### 服务的安全要求
SSL加密传输时强制要求。如果值集系统允许被实时维护，认证和审计机制是必要的。

#### 熟悉基本概念
由于术语服务是基于几个基本资源的，所以熟悉这些资源和使用方法对实现十分重要。基本资源包括：`CodeSystem`,`ValueSet`, `ConceptMap`. 以及熟悉如何在FHIR中使用编码。

#### 如何在FHIR中使用编码
FHIR中的资源很多会用到编码，这些编码往往是一串固定的值，代表一个特定的含义。FHIR这里的编码定义都是通过一对组合:`system`和`code`. `system`的值是一个url指出哪里定义了这个编码，这个值需要区分大小写。
|Key|Value|
|---|---|
|system|URI指定编码的位置|
|version|版本|
|code|字符串表示一个概念|
|display|描述文字|

编码在资源中定义时可以有多种数据类型，但都代表编码。
<table>
 <tbody><tr><td colspan="2">当编码在资源中定义时，可以有如下数据类型</td></tr>
 <tr><td>code</td><td>这数据类型只展示编码`code`, `system`隐含在元素中。</td></tr>
 <tr><td>Coding</td><td>这个数据类型定义一个标准的`code`和`system`对。</td></tr>
 <tr><td>CodeableConcept</td><td>`coding`加上一个直白的文本字段.</td></tr>
 <tr><td colspan="2">另外，非资源元素定义时，如下字段也可以带着编码.或者内容当成编码，并绑定一个值集。</td></tr>
 <tr><td>Quantity</td><td>数量这个字段可以用`system`and<i>code</i>定义单位的类型.</td></tr>
 <tr><td>string</td><td>有的时候string字符串也可以用来控制某个元素固定的几个值.</td></tr>
 <tr><td>uri</td><td>类似string, uri也可以当成编码元素</td></tr>
</tbody></table>

#### 选择编码系统
`system`对应的url需要指向一个编码系统。编码系统可以有如下地址可以引用:
- 本规范指定的编码系统仓库[code system registry](https://hl7.org/fhir/R4/terminologies-systems.html)
- 编码系统发布者定义的URI或者OID
- FHIR社区的编码系统仓库，并且状态是`active`
- 在[HL7 OID registry](http://www.hl7.org/oid/index.cfm)注册的OID

下表是所有外部编码系统：
<table>
 <tbody><tr>
   <th>URI</th>
   <th>Source</th>
   <th>Comment</th>
   <th>OID (for non-FHIR systems)</th>
 </tr>
 <tr>
   <td colspan="4" style="background: #EFEFEF"><b>Externally Published code systems</b> <a name="external"></a></td>
 </tr>

 <tr>
   <td>http://snomed.info/sct <a name="http://snomed.info/sct"></a></td>
   <td>SNOMED CT (<a href="http://snomed.org">IHTSDO <img src="external.png" style="text-align: baseline"></a>)</td>
   <td>See <a href="snomedct.html">Using SNOMED CT with FHIR</a></td>
   <td style="color : DarkGrey">2.16.840.1.113883.6.96</td>
 </tr>
 <tr>
   <td>http://www.nlm.nih.gov/research/umls/rxnorm<a name=" http://www.nlm.nih.gov/research/umls/rxnorm"></a></td>
   <td>RxNorm (<a href="http://www.nlm.nih.gov/">US NLM <img src="external.png" style="text-align: baseline"></a>)</td>
   <td>See <a href="rxnorm.html">Using RxNorm with FHIR</a></td>
   <td style="color : DarkGrey">2.16.840.1.113883.6.88</td>
 </tr>
 <tr>
   <td>http://loinc.org <a name="http://loinc.org"></a></td>
   <td>LOINC (<a href="http://loinc.org">LOINC.org <img src="external.png" style="text-align: baseline"></a>)</td>
   <td>See <a href="loinc.html">Using LOINC with FHIR</a></td>
   <td style="color : DarkGrey">2.16.840.1.113883.6.1</td>
 </tr>
 <tr>
   <td>http://unitsofmeasure.org <a name="http://unitsofmeasure.org"></a></td>
   <td>UCUM: (<a href="http://unitsofmeasure.org">UnitsOfMeasure.org <img src="external.png" style="text-align: baseline"></a>) Case Sensitive Codes</td>
   <td>See <a href="ucum.html">Using UCUM with FHIR</a></td>
   <td style="color : DarkGrey">2.16.840.1.113883.6.8</td>
 </tr>
 <tr>
   <td>http://ncimeta.nci.nih.gov <a name="http://ncimeta.nci.nih.gov"></a></td>
   <td><a href="http://ncimeta.nci.nih.gov">NCI Metathesaurus <img src="external.png" style="text-align: baseline"></a></td>
   <td>See <a href="ncimeta.html">Using NCI Metathesaurus with FHIR</a></td>
   <td style="color : DarkGrey">2.16.840.1.113883.3.26.1.2</td>
 </tr>
 <tr>
   <td>http://www.ama-assn.org/go/cpt <a name="http://www.ama-assn.org/go/cpt"></a></td>
   <td><a href="http://www.ama-assn.org/go/cpt">AMA CPT codes <img src="external.png" style="text-align: baseline"></a></td>
   <td>See <a href="cpt.html">Using CPT with FHIR</a></td>
   <td style="color : DarkGrey">2.16.840.1.113883.6.12</td>
 </tr>
 <tr>
   <td>http://hl7.org/fhir/ndfrt <a name="http://hl7.org/fhir/ndfrt"></a></td>
   <td><a href="http://www.nlm.nih.gov/research/umls/sourcereleasedocs/current/NDFRT/">NDF-RT (National Drug File – Reference Terminology) <img src="external.png" style="text-align: baseline"></a></td>
   <td>See <a href="ndfrt.html">Using NDF-RT with FHIR</a></td>
   <td style="color : DarkGrey">2.16.840.1.113883.6.209</td>
 </tr>
 <tr>
   <td>http://fdasis.nlm.nih.gov <a name="http://fdasis.nlm.nih.gov"></a></td>
   <td><a href="http://www.fda.gov/Drugs/InformationOnDrugs/ucm142438.htm">Unique Ingredient Identifier (UNII) <img src="external.png" style="text-align: baseline"></a></td>
   <td>See <a href="unii.html">Using UNII with FHIR</a></td>
   <td style="color : DarkGrey">2.16.840.1.113883.4.9</td>
 </tr>
 <tr>
   <td>http://hl7.org/fhir/sid/ndc <a name="http://hl7.org/fhir/sid/ndc"></a></td>
   <td><a href="http://www.fda.gov/Drugs/InformationOnDrugs/ucm142438.htm">NDC/NHRIC Codes <img src="external.png" style="text-align: baseline"></a></td>
   <td>See <a href="ndc.html">Using NDC with FHIR</a></td>
   <td style="color : DarkGrey">2.16.840.1.113883.6.69</td>
 </tr>
 <tr>
   <td>http://hl7.org/fhir/sid/cvx <a name="http://hl7.org/fhir/sid/cvx"></a></td>
   <td><a href="http://www2a.cdc.gov/vaccines/iis/iisstandards/vaccines.asp?rpt=cvx">CVX (Vaccine Administered) <img src="external.png" style="text-align: baseline"></a></td>
   <td>See <a href="cvx.html">Using CVX with FHIR</a></td>
   <td style="color : DarkGrey">2.16.840.1.113883.12.292</td>
 </tr>
 <tr>
   <td>urn:iso:std:iso:3166 <a name="urn:iso:std:iso:3166"></a></td>
   <td><a href="http://www.iso.org/iso/country_codes.htm">ISO Country &amp; Regional Codes <img src="external.png" style="text-align: baseline"></a></td>
   <td>See <a href="iso3166.html">Using ISO 3166 Codes with FHIR</a></td>
   <td>1.0.3166.1.2.2</td>
 </tr>
 <tr>
   <td>http://hl7.org/fhir/sid/dsm5 <a name="http://hl7.org/fhir/sid/dsm5"></a></td>
   <td><a href="https://en.wikipedia.org/wiki/DSM-5">DSM-5 <img src="external.png" style="text-align: baseline"></a></td>
   <td>Diagnostic and Statistical Manual of Mental Disorders, Fifth Edition (DSM-5)</td>
   <td>2.16.840.1.113883.6.344</td>
 </tr>
 <tr>
   <td>http://www.nubc.org/patient-discharge <a name="http://www.nubc.org/patient-discharge"></a></td>
   <td><a href="http://www.nubc.org">NUBC <img src="external.png" style="text-align: baseline"></a> code system for Patient Discharge Status</td>
   <td>National Uniform Billing Committee, manual UB-04, UB form locator 17</td>
   <td style="color : DarkGrey">2.16.840.1.113883.6.301.5</td>
 </tr>
 <tr>
   <td>http://www.radlex.org <a name="http://www.radlex.org"></a></td>
   <td><a href="http://www.radlex.org">RadLex <img src="external.png" style="text-align: baseline"></a></td>
   <td>(Includes <a name="http://playbook.radlex.org">play book</a> codes)</td>
   <td style="color : DarkGrey">2.16.840.1.113883.6.256 </td>
 </tr>
 <tr>
   <td>
     ICD-9, ICD-10
   </td>
   <td><a href="http://www.who.int/classifications/icd/en/">WHO <img src="external.png" style="text-align: baseline"></a>) &amp; National Variants</td>
   <td>See <a href="icd.html">Using ICD-[x] with FHIR</a></td>
   <td>
     See ICD page for details
   </td>
 </tr>
 <tr>
   <td>
     http://hl7.org/fhir/sid/icpc-1 <a name="http://hl7.org/fhir/sid/icpc-1"></a> <br>
     http://hl7.org/fhir/sid/icpc-1-nl <a name="http://hl7.org/fhir/sid/icpc-1-nl"></a> <br>
     http://hl7.org/fhir/sid/icpc-2 <a name="http://hl7.org/fhir/sid/icpc-2"></a>
   </td>
   <td>ICPC (International Classification of Primary Care) (<a href="http://www.ph3c.org/">PH3C <img src="external.png" style="text-align: baseline"></a>)</td>
   <td>
     <br>
     <a href="https://referentiemodel.nhg.org/tabellen/nhg-tabel-24-icpc1">NHG Table 24 ICPC-1 (NL) <img src="external.png" style="text-align: baseline"></a><br>
   </td>
   <td>
      <br>
     2.16.840.1.113883.2.4.4.31.1 <br>
     2.16.840.1.113883.6.139
   </td>
 </tr>
 <tr>
   <td>http://hl7.org/fhir/sid/icf-nl <a name="http://hl7.org/fhir/sid/icf-nl"></a></td>
   <td>ICF  (International Classification of Functioning, Disability and Health) (<a href="http://www.who.int/classifications/icf/en/">WHO <img src="external.png" style="text-align: baseline"></a>)</td>
   <td></td>
   <td style="color : DarkGrey">2.16.840.1.113883.6.254</td>
 </tr>
 <tr>
   <td>http://terminology.hl7.org/CodeSystem/v2-[X](/v) <a name="http://terminology.hl7.org/CodeSystem/v2-[X](/v"></a></td>
   <td><a href="terminologies-v2.html">Version 2 tables</a></td>
   <td>[X] is the 4 digit identifier for a table; e.g. http://terminology.hl7.org/CodeSystem/v2-0203<br>
     Note: only <a href="terminologies-v2.html">some tables</a> may be treated in this fashion.
     For some tables, the meaning of the code is version dependent, and so additional information
     must be included in the namespace, e.g. http://terminology.hl7.org/CodeSystem/v2-0123/2.3+, as defined in the <a href="terminologies-v2.html">v2 table namespace list</a>.
     Version 2 codes are case sensitive.</td>
   <td style="color : DarkGrey">2.16.840.1.113883.12.[X]</td>
 </tr>
 <tr>
   <td>http://terminology.hl7.org/CodeSystem/v3-[X] <a name="http://terminology.hl7.org/CodeSystem/v3-[X"></a></td>
   <td><a href="terminologies-v3.html">A </a><a href="https://www.hl7.org/implement/standards/product_brief.cfm?product_id=186">HL7 v3 <img src="external.png" style="text-align: baseline"></a> code system</td>
   <td>[X] is the code system name; e.g. http://terminology.hl7.org/CodeSystem/v3-GenderStatus. HL7 v3 code systems are case sensitive.</td>
   <td>see <a href="terminologies-v3.html">v3 list</a></td>
 </tr>
 <tr>
   <td>https://www.gs1.org/gtin <a name="https://www.gs1.org/gtin"></a></td>
   <td>GTIN (<a href="https://www.gs1.org">GS1 <img src="external.png" style="text-align: baseline"></a>)</td>
   <td>Note: GTINs may be used in both <a href="datatypes.html#Coding">Codes</a> and <a href="datatypes.html#Identifier">Identifiers</a></td>
   <td style="color : DarkGrey">1.3.160</td>
 </tr>
 <tr>
   <td>http://www.whocc.no/atc <a name="http://www.whocc.no/atc"></a></td>
   <td>Anatomical Therapeutic Chemical Classification System (<a href="http://www.whocc.no/atc/structure_and_principles/">WHO <img src="external.png" style="text-align: baseline"></a>)</td>
   <td></td>
   <td style="color : DarkGrey">2.16.840.1.113883.6.73</td>
 </tr>
 <tr>
   <td>urn:ietf:bcp:47 <a name="urn:ietf:bcp:47"></a></td>
   <td>IETF language (see <a href="http://tools.ietf.org/html/bcp47">Tags for Identifying Languages - BCP 47 <img src="external.png" style="text-align: baseline"></a>)</td>
   <td>This is used for identifying language throughout FHIR. Note that usually these codes are in a <code>code</code> and the system is assumed</td>
   <td></td>
 </tr>
 <tr>
   <td>urn:ietf:bcp:13 <a name="urn:ietf:bcp:47"></a></td>
   <td>Mime Types (see <a href="http://tools.ietf.org/html/bcp13">Multipurpose Internet Mail Extensions (MIME) Part Four - BCP 13 <img src="external.png" style="text-align: baseline"></a>)</td>
   <td>This is used for identifying the mime type system throughout FHIR. Note that these codes are in a <code>code</code> (e.g. <a href="datatypes.html#Attachment">Attachment.contentType</a>
   and in these elements the system is assumed). This system is defined for when constructing value sets of mime type codes</td>
   <td></td>
 </tr>
 <tr>
   <td>urn:iso:std:iso:11073:10101 <a name="urn:iso:std:iso:11073:10101"></a></td>
   <td>Medical Device Codes (<a href="https://www.iso.org/standard/37890.html">ISO 11073-10101 <img src="external.png" style="text-align: baseline"></a>)</td>
   <td>See <a href="mdc.html">Using MDC Codes with FHIR</a></td>
   <td style="color : DarkGrey">2.16.840.1.113883.6.24</td>
 </tr>
 <tr>
   <td><a href="codesystem-dicom-dcim.html">http://dicom.nema.org/resources/ontology/DCM</a> <a name="http://dicom.nema.org/resources/ontology/DCM"></a></td>
   <td>DICOM Code Definitions</td>
   <td>The meanings of codes defined in DICOM, either explicitly or by reference to another part of DICOM or an external reference document or standard</td>
   <td>1.2.840.10008.2.16.4</td>
 </tr>
 <tr>
   <td>http://hl7.org/fhir/NamingSystem/ca-hc-din <a name="http://hl7.org/fhir/NamingSystem/ca-hc-din"></a></td>
   <td><a href="http://www.hc-sc.gc.ca/dhp-mps/prodpharma/activit/fs-fi/dinfs_fd-eng.php">Health Canada Drug Identification Number <img src="external.png" style="text-align: baseline"></a></td>
   <td>
     <p>A computer-generated eight-digit number assigned by Health Canada to a drug product prior to being marketed in Canada.
       <a href="http://www.hc-sc.gc.ca/dhp-mps/prodpharma/databasdon/index-eng.php">Canada Health Drug Product Database <img src="external.png" style="text-align: baseline"></a> contains
       product specific information on drugs approved for use in Canada.
      </p>
    </td>
   <td style="color : DarkGrey">2.16.840.1.113883.5.1105</td>
 </tr>
 <tr>
   <td>http://hl7.org/fhir/sid/ca-hc-npn <a name="http://hl7.org/fhir/NamingSystem/ca-hc-npn"></a></td>
   <td><a href="https://www.canada.ca/en/health-canada/services/drugs-health-products/natural-non-prescription/applications-submissions/product-licensing/licensed-natural-health-products-database.html">Health Canada Natural Product Number <img src="external.png" style="text-align: baseline"></a></td>
   <td>
     <p>A computer-generated number assigned by Health Canada to a natural health product prior to being marketed in Canada.
      </p>
    </td>
   <td style="color : DarkGrey">2.16.840.1.113883.5.1105</td>
 </tr>
 <tr>
   <td>http://nucc.org/provider-taxonomy <a name="http://nucc.org/provider-taxonomy"></a></td>
   <td><a href="http://www.nucc.org/index.php/code-sets-mainmenu-41/provider-taxonomy-mainmenu-40/csv-mainmenu-57">NUCC Provider Taxonomy <img src="external.png" style="text-align: baseline"></a></td>
   <td>
     <p>The Health Care Provider Taxonomy code is a unique alphanumeric code, ten characters in length. The code set is structured into
     three distinct "Levels" including Provider Type, Classification, and Area of Specialization.</p>
     <p>Copyright statement for NUCC value sets: </p>
     <blockquote>
     This value set includes content from NUCC Health Care Provider Taxonomy Code Set for providers which is copyright © 2016+ American Medical Association. For commercial use, including sales or licensing, a license must be obtained
     </blockquote>
   </td>
   <td style="color : DarkGrey">2.16.840.1.113883.6.101</td>
 </tr>
 <tr>
   <td colspan="4" style="background: #EFEFEF"><b>Code Systems for Genetics</b> <a name="genetics"></a></td>
 </tr>
 <tr>
   <td>http://www.genenames.org<a name="http://www.genenames.org"> </a></td>
   <td><a href="http://www.genenames.org">HGNC: Human Gene Nomenclature Committee <img src="external.png" style="text-align: baseline"></a></td>
   <td></td>
   <td style="color : DarkGrey">2.16.840.1.113883.6.281</td>
 </tr>
 <tr>
   <td>http://www.ensembl.org<a name="http://www.ensembl.org"> </a></td>
   <td><a href="http://www.ensembl.org">ENSEMBL reference sequence identifiers <img src="external.png" style="text-align: baseline"></a></td>
   <td>Maintained jointly by the European Bioinformatics Institute and Welcome Trust Sanger Institute</td>
   <td style="color: #777777"><i>not assigned yet</i></td>
 </tr>
 <tr>
   <td>http://www.ncbi.nlm.nih.gov/refseq<a name="http://www.ncbi.nlm.nih.gov/refseq"> </a></td>
   <td><a href="https://www.ncbi.nlm.nih.gov/refseq">RefSeq: National Center for Biotechnology Information (NCBI) Reference Sequences <img src="external.png" style="text-align: baseline"></a></td>
   <td></td>
   <td style="color : DarkGrey">2.16.840.1.113883.6.280</td>
 </tr>
 <tr>
   <td>http://www.ncbi.nlm.nih.gov/clinvar<a name="http://www.ncbi.nlm.nih.gov/clinvar"> </a></td>
   <td><a href="http://www.ncbi.nlm.nih.gov/clinvar">ClinVar Variant ID <img src="external.png" style="text-align: baseline"></a></td>
   <td>NCBI central repository for curating pathogenicity of potentially clinically relevant variants</td>
   <td style="color: #777777"><i>not assigned yet</i></td>
 </tr>
 <tr>
   <td>http://sequenceontology.org<a name="http://sequenceontology.org"> </a></td>
   <td><a href="http://sequenceontology.org">Sequence Ontology <img src="external.png" style="text-align: baseline"></a></td>
   <td></td>
   <td style="color: #777777"><i>not assigned yet</i></td>
 </tr>
 <tr>
   <td>http://varnomen.hgvs.org<a name="http://varnomen.hgvs.org"> </a></td>
   <td><a href="http://varnomen.hgvs.org">HGVS : Human Genome Variation Society  <img src="external.png" style="text-align: baseline"></a></td>
   <td></td>
   <td style="color : DarkGrey">2.16.840.1.113883.6.282</td>
 </tr>
 <tr>
   <td>http://www.ncbi.nlm.nih.gov/projects/SNP<a name="http://www.ncbi.nlm.nih.gov/projects/SNP"> </a></td>
   <td><a href="http://www.ncbi.nlm.nih.gov/projects/SNP">DBSNP : Single Nucleotide Polymorphism database <img src="external.png" style="text-align: baseline"></a></td>
   <td></td>
   <td style="color : DarkGrey">2.16.840.1.113883.6.284</td>
 </tr>
 <tr>
   <td>http://cancer.sanger.ac.uk/<br>cancergenome/projects/cosmic<a name="http://cancer.sanger.ac.uk/cancergenome/projects/cosmic"> </a></td>
   <td><a href="http://cancer.sanger.ac.uk/cancergenome/projects/cosmic">COSMIC : Catalogue Of Somatic Mutations In Cancer <img src="external.png" style="text-align: baseline"></a></td>
   <td></td>
   <td style="color : DarkGrey">2.16.840.1.113883.3.912</td>
 </tr>
 <tr>
   <td>http://www.lrg-sequence.org<a name="http://www.lrg-sequence.org"> </a></td>
   <td><a href="http://www.lrg-sequence.org">LRG : Locus Reference Genomic Sequences <img src="external.png" style="text-align: baseline"></a></td>
   <td></td>
   <td style="color : DarkGrey">2.16.840.1.113883.6.283</td>
 </tr>
 <tr>
   <td>http://www.omim.org<a name="http://www.omim.org"> </a></td>
   <td><a href="http://www.omim.org">OMIM : Online Mendelian Inheritance in Man <img src="external.png" style="text-align: baseline"></a></td>
   <td></td>
   <td style="color : DarkGrey">2.16.840.1.113883.6.174</td>
 </tr>
 <tr>
   <td>http://www.ncbi.nlm.nih.gov/pubmed<a name="http://www.ncbi.nlm.nih.gov/pubmed"> </a></td>
   <td><a href="http://www.ncbi.nlm.nih.gov/pubmed">PubMed  <img src="external.png" style="text-align: baseline"></a></td>
   <td></td>
   <td style="color : DarkGrey">2.16.840.1.113883.13.191</td>
 </tr>
 <tr>
   <td>http://www.pharmgkb.org<a name="http://www.pharmgkb.org"> </a></td>
   <td><a href="http://www.pharmgkb.org">PHARMGKB : Pharmacogenomic Knowledge Base <img src="external.png" style="text-align: baseline"></a></td>
   <td>PharmGKB Accession ID</td>
   <td style="color : DarkGrey">2.16.840.1.113883.3.913</td>
 </tr>
 <tr>
   <td>http://clinicaltrials.gov<a name="http://clinicaltrials.gov"> </a></td>
   <td><a href="http://clinicaltrials.gov">ClinicalTrials.gov <img src="external.png" style="text-align: baseline"></a></td>
   <td></td>
   <td style="color : DarkGrey">2.16.840.1.113883.3.1077</td>
 </tr>
 <tr>
   <td>http://www.ebi.ac.uk/ipd/imgt/hla <a name="http://www.ebi.ac.uk/ipd/imgt/hla"></a></td>
   <td><a href="http://www.ebi.ac.uk/ipd/imgt/hla">European Bioinformatics Institute <img src="external.png" style="text-align: baseline"></a></td>
   <td></td>
   <td style="color : DarkGrey">2.16.840.1.113883.6.341</td>
 </tr>
</tbody></table>