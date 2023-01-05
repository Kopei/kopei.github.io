---
layout: post
title: Eyes on FHIR
categories: [standard]
description: 
keywords: ophathalmology, HL7, FHIR
catalog: true
multilingual: false
tags: FHIR, Standard
date: 2022-11-20
---

# 什么是 Eyes on FHIR ？

> 从事医疗信息化的朋友可能听说过FHIR(Fast Healthcare Interoperability Resouce), 但是在具体实现和应用上可能没有相关经验。下面这篇文章里，我们一起来看看美国的一个Patient Care project在眼科是怎么落地FHIR的。

## 背景

2020年美国《21世纪医疗法》最终法规要求健康技术供应商必须支持2个公开的API能够使患者访问自己的健康数据。这两个API需要遵循HL7 FHIR标准。随着社区力量的驱动，FHIR的流行度和成熟度在美国逐渐成型，使得健康信息数据互操作性得到了很大的提升。


## Eyes on FHIR的目的

虽然在美国FHIR的发展有很大进展，但是真实世界项目的落地尤其是临床专科的应用仍十分有限，主要原因是没有统一的实施指引（Implementation Guidance)。所以这个项目的目标就是开发必要的国际标准使FHIR能够真正地给某个专科带来价值。此项目的实施方式主要通过把眼科尽量全面的临床词典映射到FHIR格式，最后形成实施标准来帮助实现系统间的互联互通。

![](https://confluence.hl7.org/download/attachments/82914199/image2021-8-7_14-15-43.png?version=1&modificationDate=1628309743891&api=v2)

具体流程如下：

1. 列出一系列眼科真实世界的用例，尤其是有互操作性痛点的案例。用例的形式需要考虑如下情形：
	- 涉及的元素
		- Actors 操作人
    	- Pre-condition 先决条件
    	- Workflow 工作流
    	- Post-condition 后置条件
    	- Exceptions 例外
    	- Data elements 数据元素
    	- Necessary commentary 必要的备注
     - 专用词汇的定义
     	- FHIR 资源
        - 临床专业术语， 如SNOMED-CT, LOINC, DICOM
     - 局限和差距的发现
     	- 临床上，在某些地方的设备代码并不全面
        - 如何做FHIR profile

2. 收集和整理如上信息，包括编码系统等。
3. 把上述信息转化为FHIR Profiles, 配置是否是必须项。
4. 开发实施指引IG, 甄别如何使用FHIR资源解决某个特定的互操作性问题。
5. 发布实施指引
6. 参加三年一次的技术连接会，演示用例和测试互操作性。
7. 参加三年一次的HL7-FHIR投票，获准正式发布实施指引。
8. 转化IG, 让更多真实世界的案例可以使用此指引。

## 项目预期结果
- 建立治理、质控和伦理的原则；促成眼科相关各界人事代表组成委员会共同发展未来路线图。
- 临床和研究影响力。作为第一个临床专科实施项目，藉此机会展示FHIR的临床好处；记录可以重复利用的技术、协议、流程等材料。

## 此项目使用的场景

1. 日常临床关护沟通相关；病人旅程相关，比如：纵向病患旅程；异步通信；远程医疗设备等
2. 医师临床工具， 如SMART app/CDS hooks/integrating remote monitoring
3. 患者注册数据收集， 如批量数据导出
4. 临床工作流优化，如自动支付、审计、优先授权、行政管理。
5. 辅助生命科学和研发，如辅助临床招募，数据统一。
6. AI图像诊断工作流，如下图：

![](https://confluence.hl7.org/download/attachments/82914199/image2021-8-8_10-3-20.png?version=1&modificationDate=1628381000230&api=v2)

## 具体细节功能
1. Providing guidance for the FHIR representation of a comprehensive set of codified **clinical findings** to facilitate interoperable exchange of these data elements between systems.
2. Providing guidance for the FHIR representation of a comprehensive set of codified **diagnoses** to facilitate interoperable exchange of these data elements between systems.
3. Providing guidance for the FHIR representation of a comprehensive set of codified **retinal therapeutics** to facilitate interoperable exchange of these data elements between systems.
4. Exchanging information between and EHR (/PMS) and a diagnostic device where:
EHR Supports DICOM Modality Worklist, Storage and Display (With **no PACS**)
Based on IHE's Unified Eyecare Workflow 'real world model' (RWM) II
5. Exchanging information between and EHR (/PMS) and a diagnostic device where:
EHR Supports DICOM Modality Worklist and Integrates with a **PACS**
Based on IHE's Unified Eyecare Workflow 'real world model' (RWM) I
6. Exchanging information between and EHR (/PMS) and a diagnostic device where:
EHR Implements HL7 Only (**no DICOM** support) and Integrates with a PACS)
Based on IHE's Unified Eyecare Workflow 'real world model' (RWM) III
7. Sending a referral containing clinical information from and/or to any combination of the following practitioners: ophthalmologist, optometrist, general practitioner, specialist
8. Sending a referral from a clinician to a healthcare service (eg - an ambulatory service centre when booking for cataract surgery.
9. Exchanging information between and EHR and a clinical registry (eg - FRB!) - where:
The registry is highly structured；Not all registry required data points are routinely captured in a structured format in the EHR；Patient details must be de-identified.
10. Sending in bulk either all or part of an entire clinical record to a clinical registry.
11. Sending select data elements from select patients (de-identified) in bulk to a research institute / life science body.
12. The **referral** of a patient to a clinical trial.
13. Automating prior authorization for anti-VEGF injections.
14. Real time clinical trial recruitment using **CDS** hooks (eg for treatment-naive wet-AMD)
15. SMART on FHIR app to compile and display relevant information into a single screen / application


#### 外部链接
- [Eyes on FHIR](https://confluence.hl7.org/pages/viewpage.action?pageId=82914199)