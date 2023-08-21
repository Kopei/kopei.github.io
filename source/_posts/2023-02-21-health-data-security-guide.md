---
title: Health Data Security Guide
comment: true
date: 2023-02-21 10:56:17
tags: Data security
---
> This article is a translation of Chinese national standard GB/T39725-2020.

## 0. Introduction
Health data includes personal health data and health related data obtained from processed system. With the vigorous development of health application, "Internet + health" and intelligent medical care, various new businesses and applications are emerging, and health data are facing more and more security challenges in all stages of the life cycle, and security issues are frequently occurring. Since the security of health data is related to the safety of patients' lives, personal information security, social public interest and national security, in order to better protect the security of health data, regulate and promote the integration and sharing of health data, development and application, and promote the development of health business, this health data security guideline is formulated. 

## 1. Scope
This standard gives the security measures that health data controllers can take when protecting health data. 
This standard is suitable for guiding health data controllers in the security protection of health data, and can also be used as a reference for health, cyber security-related authorities and third-party assessment organizations to carry out security supervision, management and assessment of health data. 

## 2. Normative Reference 
The following documents are essential to the application of this document. Where a reference document is dated, only the dated version applies to this document. Where the reference document is not dated, its latest version (including all the change orders) applies to this document. 
|National standard document code	     |Category       |Document Name       |Applicable|
| --- | --- | --- | --- |
|  GB/T 22080—2016      |  Information Security tech     |   Information Security Management System      |Required|
|GB/T 22081—2016	|Information Security tech|Information Security Control Practice Guide|Guide|
|GB/T 22239—2019|Information Security tech|Baseline for Classified protection of cybersecurity|Required|
|GB/T 25069|Information Security tech|Terminology||
|GB/T 31168|Information Security tech|Security capability requirements of cloud computing services|Required|
|GB/T 35273|Information Security tech	|Personal information security specification|Guide|
|GB/T 35274—2017|Information Security tech|Security capability requirements for big data services|Required|
|GB/T 37964—2019|Information Security tech|Guide for de-identifying personal information|Guide|
|ISO 80001||Application of risk management for IT-networks incorporating medical devices	 ||

## 3. Terms & Definition
### 3.1 Personal health data
Electronic data that, alone or in combination with other information, can identify a specific natural person or reflect the physical or mental health of a specific natural person Data.
Note: Personal health data includes an individual's past, present or future physical or mental health, received health services and health services paid for, etc. See Appendix A.

### 3.2 Health data
Personal health data and health-related electronic data obtained by processing 
Examples: Overall group analysis results, trend forecasts, disease prevention and control statistics, etc., obtained after processing group health data.

### 3.3 Health service professional
A person authorized by a government or industry organization to be qualified to perform specific health work.
Example: Doctors.

### 3.4 Health service
Services provided by a health professional or paraprofessional that have an impact on a health condition.

### 3.5 Health data controller
An organization or individual who can determine the purpose, manner, and scope of health data processing, etc.
Example: Organizations that provide health services, health insurance agencies, government agencies, health scientific research institutions, individual clinics, etc.

### 3.6 Health information system
A system for capturing, storing, processing, transmitting, accessing, and destroying health data in a computed form.

### 3.7 Limited data set
Personal health data sets that have been partially de-identified but still are able to identify the corresponding individual and therefore need to be protected.
Example: Removal of identifiers directly related to the individual and his or her dependents, family members, and employers from health data.
Note: Limited datasets may be used for scientific research, medical/health education, and public health purposes without the authorization of the individual.

### 3.8 Notes of treatment
Observations, reflections, treatment, discussions, and conclusions recorded by health professionals in the course of providing health services.
Note: Treatment notes have intellectual property attributes and are the property of the health professional and/or his or her organization.

### 3.9 Disclosure
The act of transferring and sharing health data to specific individuals or organizations, as well as publicly releasing it to unspecified individuals, organizations or society.

### 3.10 Clinical research
Scientific research activities aimed at exploring the causes, prevention, diagnosis, treatment and prognosis of diseases, initiated by medical institutions, academic research institutions and/or medical and health-related companies, with patients or healthy people as research subjects.
Note: Clinical research belongs to a branch of medical research.

### 3.11 Completely public sharing
Once data is released, it is difficult to recall and is generally released directly and publicly via the Internet. See [GB/T37964—2019, 3.12].

### 3.12 Controlled public sharing
Control the use of data through data use agreements. See [GB/T37964—2019, 3.13].

### 3.13 Enclave public sharing
Shared within the physical or virtual territory, data cannot flow outside the territory. See [BG/T37964—2019, 3.14].


## 4. Abbreviations.
Following are the abbreviations.

- ACL: （Access Control Lists）
- API: （Application Programming Interface）
- APP: （Application）
- DNA: （Deoxyribonucleic Acid）
- EDC:  (Electronic Data Capture)
- GCP：（Good Clinical Practice）
- HIS: （Hospital Information System）
- HIV: （Human Immunodeficiency Virus） 
- HL7: （Health Level Seven）
- ID：（Identity）
- IP: （Internet Protocol）
- IPSEC：（Internet Protocol Security）
- LDS：（Limited Data Set Files）
- PIN: （Personal Identity Number） 
- PUF：（Public Use Files）
- RIF: （Research Identifiable Files）
- RNA: （Ribonucleic Acid）
- SQL: （Structured Query Languages）
- TLS：（Transport Layer Security）
- USB：（Universal Serial Bus）
- VPN: （Virtual Private Network）
- XSS：（Cross-site scripting）

## 5. Security Target
Health data controllers are advised to adopt reasonable and appropriate management and technical safeguards to achieve the following objectives
a) Ensure the confidentiality, integrity and availability of health data.
b) Ensure the legality and compliance of the health data use and disclosure process to protect the security of personal information, public interest and national security.
c) Ensure that health data meets business development needs while meeting the above security requirements.

## ６. Classification
### 6.1  Data Category
Health data can be classified as
a) Personal attribute, is data that alone or in combination with other information, can identify a specific natural person.
b) Health status, is data that reflects or is closely related to the health condition of an individual. 
c) Medical application data, is data that reflects medical care, outpatient, inpatient, discharge, and other medical services.
d) Health payment data, are data related to the costs involved in services such as health or insurance.
e) Health resource data, are those data that capture the capacity and characteristics of health providers, health programs, and health systems.
f) Public health data, are data related to public utilities that are relevant to the public health of a country or region.

The specific content of each type of data is shown in Table 1. The data elements, data sets, value codes and other related standards used in the health information domain can be found in Appendix B.

<center>Table 1</center>

|Data Category|Scope|
| --- | --- |
|Personal attribute	|1) Demographic information, including name, date of birth, gender, ethnicity, nationality, occupation, address, employer, family member information, contact information, income, marital status, etc. 2) personal identification information, including name, ID card, work permit, residence permit, social security card, images that can identify the individual, health card number, hospitalization number, various types of examination and test-related bill numbers, etc. 3) personal communication information, including personal telephone numbers, mailboxes, account numbers and associated information, etc. 4) personal biometric information, including genetic, fingerprint, voice print, palm print, ear, iris, facial features, etc. 5) personal health monitoring sensor ID, etc.|
|Health status  | Chief complaint, current and past medical history, physical examination (signs), family history, symptoms, test and examination data, genetic counseling data, health-related data collected by wearable devices, lifestyle, gene sequencing, transcription product sequencing, protein analysis and measurement, metabolic small molecule testing, human microbial testing, etc.  |
| Medical application data |Outpatient (emergency) medical records, inpatient medical prescriptions, examination and test reports, medication information, course records, surgery records, anesthesia records, blood transfusion records, nursing records, admission records, discharge summaries, referral (hospital) records, informed information, etc.   |
| Medical payment data | 1) Medical transaction information, including medical insurance payment information, transaction amount, transaction records, etc. 2）Insurance information, including insurance status, insurance amount, etc. |
| Health Resource data |  Basic hospital information, hospital operation data, etc.|
| Public health data | Environmental health data, epidemic disease data, disease monitoring data, disease prevention data, birth and death data, etc.  |

### 6.2 Data Grading
Health data can be classified into the following five levels based on the level of data importance, risk level, and the level of possible damage and impact on the individual.
(a) Level 1: Data that can be used completely public. This includes data that can be accessed through public channels, such as hospital names, addresses, telephone numbers, etc., which can be directly disclosed to the public on the Internet.
(b) Level 2: Data that can be accessed on a large scale. For example, data that cannot identify individuals and can be used for research and analysis by doctors in each department upon application for approval.
(c) Level 3: Data that can be accessed on a medium scale and may cause moderate damage to individual subjects if disclosed without authorization. For example, data that has been partially de-identified, but may still be re-identified, is limited to use within the scope of the authorized project team. 
d) Level 4: Data that is available for access use on a small scale and may cause a high degree of harm to the individual if disclosed without authorization. For example, data that can directly identify an individual is restricted to access and use by health professionals involved in healthcare activities.
(e) Level 5: Data that is accessible only to a very small extent and under strictly limited conditions, which, if disclosed without authorization, may cause serious damage to the individual subject. For example, details of specific diseases (e.g., AIDS, STDs) are restricted to access by treating health professionals and require strict controls.

### 6.3 Roles Classification
For a data-specific scenario, the relevant organizations or individuals can be classified into the following four types of roles. Any organization or individual can only be classified into one of these roles for a specific data, a specific scenario or a specific data processing behavior.
(a) Personal health data subject (hereinafter referred to as "subject"): the natural person identified by personal health data. 
(b) Health data controller (hereinafter referred to as "controller"): See Definition 3.5. To decide whether an organization or individual can determine the purpose, manner, and scope of health data processing, it is possible to consider
    - 1) Whether the processing of health data is necessary for the organization or individual to comply with a law or regulation.
    - 2) whether the health data processing is necessary for the organization or individual to perform its public functions.
    - 3) whether the health data processing is decided by the organization or individual itself or jointly with other organizations or individuals.
    - 4) Whether the health data processing is authorized by the individual or by the government authorize to the organization or the individual.
    
The organization or individual who jointly decides the purpose, manner, and scope of a data processing is the joint controller.
(c) Health data processor (hereinafter referred to as "processor"): An organization or individual that collects, transmits, stores, uses, processes or discloses health data in its possession on behalf of the controller, or provides the controller with services related to the use, processing or disclosure of health data. Common processors include: health information system providers, health data analysis companies, and assisted treatment solution providers.
(d) Health data users (hereinafter referred to as "users"): Organizations or individuals who do not belong to the subjects, controllers or processors, but make use of health data for specific scenarios of specific data.

### 6.4  Data Flow Scenarios
Based on the data flow between different roles, the data exchange usage scenarios can be divided into the following six types, as shown in Figure 1.
a) Subjects->controller data flow.
b) Controller->subjects data flow. 
c) Controller<->controller intra data flow. 
d) Controller<->processor data exchange. 
e) Inter-controller data exchange.
f) Controller -> users data flow.

![](https://fastly.jsdelivr.net/gh/filess/img15@main/2023/02/20/1676885763779-aad14d99-2adb-4d70-9dde-cbe4469005f2.png)
<center>Figure 1 </center>

### 6.5 Data Public Ways
The types of data public can be divided into completely public sharing, controlled public sharing, and enclave public sharing, which correspond to different de-identification requirements and are handled by the regulations of GB/T37964--2019. Common forms of data openness and their applicable types of sharing are shown in Table 2.

|  Ways of open | Description | Applicable type of sharing |
| --- | --- | --- |
|   Public website    | Statistical aggregated data or anonymized data are available to the public and can be downloaded and analyzed by themselves.      |  Completely public     |
| File share      | Generated files from data systems and push them to SFTP interface devices or applications, or share them using mobile media      |  Controlled public     |
| API interface      | Data is provided between systems through request-response, with the data system providing real-time or quasi-real-time data service application interfaces to specific users, with the demand-side system initiating the request and the data system returning the required data, e.g. through the Webservice interface	      | Controlled public      |
| Online query      |   Query the web page provided by the data system for relevant information	    |  Completely public sharing of data (anonymous queries). Controlled public sharing (user query)     |
|  Data analysis platform|   The platform provides system environment, analytical mining tools, and de-identified sample data or simulation data. Platform users use shared or dedicated hardware and data resources, can deploy their own data and data analysis algorithms, and can query the data and analysis results within their authority. All the original data of the platform cannot be exported; the output and download of analysis results must be approved by the approval before they can be exported to the public    |    enclave public sharing|

## 7. Use and Disclosure Principles
It is appropriate for the controller to follow the below principles in the use or disclosure of health data.
- a) When using or disclosing personal health data, it is appropriate for the controller to inform the subject and obtain the subject's authorization (except in the case of b below); all communications are appropriate in plain language and contain specific information about the content of the data to be disclosed or used, the recipient of the data, the purpose of the data and how it will be used, the duration of the use of the data, the rights of the data subject, and the protective measures taken by the controller. The use or disclosure of personal health data must not exceed the scope of the individual's authorization. If it is necessary to use or disclose beyond the scope due to business needs, it is appropriate to obtain the consent of the subject again.
- b) A controller may use or disclose personal health data without the subject's authorization in the following cases.
	1) When providing the subject with his or her own health data.
	2) When treatment, payment or health service is provided.
	3) When required by public interest or laws and regulations
	4) when the limited data set is used for scientific research, medical/health education, or public health purposes.
In the above cases, the controller may rely on legal and regulatory requirements, professional ethics, ethical and professional judgment to determine which personal health data are allowed to be used or disclosed.
- c) It is appropriate for the controller to obtain authorization from the subject to use or disclose personal health data for marketing activities, except for face-to-face marketing communications between the controller and the subject. It is appropriate for the authorization for marketing activities to be communicated to the subject in a reasonable manner and for the subject to be fully informed and to give explicit and autonomous consent. The authorization should be independent and should not be a precondition for the subject to obtain any public service or medical service or bundled with other service terms. The controller is advised to inform the subject in writing, at the same time as obtaining the authorization, of its right to revoke the authorization at any time.
- d) The subject (or his authorized representative) has the right to access his personal health data or to request disclosure of his data, and the controller is advised to disclose the corresponding personal health data upon his request.
- e) The subject has the right to review and obtain a copy of his or her personal health data, which the controller may provide, for example, through file sharing or online access.
- f) If the subject discovers that the controller holds inaccurate or incomplete personal health data about the subject, the controller is encouraged to provide the subject with the means to request correction or additional information.
- g) The Subject has the right to a historical review of the use or disclosure of data by the Controller or its processors for a minimum period of six years.
- h) The Subject has the right to request the Controller to restrict the use or disclosure of its personal health data in the course of diagnosis, treatment, payment, health services, etc., as well as to restrict the disclosure of information to related persons, and the Controller is not obliged to agree to such restriction requests; however, once agreed, it is appropriate for the Controller to comply with the agreed restrictions unless required by law or regulation and in medical emergencies.
- i) Controllers may use treatment notes for therapeutic purposes and, after necessary de-identification, may use or disclose treatment notes for internal training and academic seminars without individual authorization.
- j) It is appropriate for the controller to develop and implement reasonable strategies and processes to limit use and disclosure to a minimum.
- k) Controllers are advised to confirm that the security capabilities of the processor meet the security requirements and to sign a data processing agreement before allowing the processor to perform data processing for them. Processors are advised to process data in accordance with the controller's requirements, and processors cannot bring in third parties to assist in data processing without the controller's permission.
- l）Before the controller provides the data to the third party controller authorized by the government, it is appropriate to obtain the relevant documents with the official seal of the government, and after the data is provided, the responsibility for data security and the security of the transmission channel shall be borne by the third party controller.
- m) The controller may use the limited data set for scientific research, healthcare services, public health and other purposes after confirming the legality, legitimacy and necessity of data use and that the user has the appropriate data security capabilities, and the user has signed a data use agreement and committed to protecting the personal healthcare data in the limited data set; the user may only use the data within the scope agreed upon in the agreement and assume the responsibility for data security, and after the use of the data is completed, it is appropriate to return, completely destroy or otherwise dispose of the data in accordance with the controller's requirements. Users may not disclose data to third parties without the permission of the controller.
- n) If the controller obtains health-related data that does not identify the individual after aggregation and analysis of personal health data, the data is no longer personal information, but its use and disclosure are subject to other relevant national regulatory requirements.
o) If the controller needs to provide the corresponding data outside of China for academic research, after the necessary de-identification process and after discussion and approval by the Data Security Committee, non-confidential and non-important data within 250 items can be provided, otherwise it is appropriate to submit it to the relevant department for approval.
- p) Data that does not involve state secrets, important data or other data that are prohibited or restricted from being provided outside the country, the controller may provide personal health data to overseas destinations with the consent of the subject's authorization and the consent of the Data Security Committee for discussion and approval, and it is appropriate to control the cumulative amount of data within 250 items, otherwise it is appropriate to submit it to the relevant department for approval.
- q) It is not appropriate for the controller to store health data in servers outside the country, and not to host or lease servers outside the country.
- r) When the controller cooperates in data development and utilization, it is appropriate to adopt the open form of "data analysis platform" and strictly control the disclosure of data use.

## 8. Key Safety Measures
### 8.1 Graded Key Safety Measures
Data grading can be carried out according to the needs of data protection, and different security protection measures can be implemented for different levels of data, focusing on authorization management, identity identification, and access control management. For example, the data grading and security measures from the perspective of personal information security risk are shown in Table 3. The data grading and security measures under the scenario of doctors access are shown in 11.1. The data grading and security measures under the scenario of clinical research are shown in 11.3.

Table 3. Highlights of data grading and security measures from personal information security risks

|  Data grading     |   Business requirements，content & users |  Scenarios       |  Features and examples     |    Key safety measures   |
| --- | --- | --- | --- | --- |
|  Level 1	     |Business requirements: data can be publicly released;Data Content: Certain statistical data;Users: The general public       | Public announcement	      | For example, information on remaining inpatient beds, remaining available outpatient numbers	      |  Need approval when public announces     |
|    Level 2   | Business requirements: no need to identify individuals; Data content: general demographic information, various types of medical and health service information; Data users: scientific research and education and others      |   Management, Research, Education and Statistical Analysis    | No need to identify individuals, e.g., case analysis, various types of disease distribution statistics, epidemiological studies, disease cohort studies, etc. Examples of scenarios: clinical research, medical health education, drug/medical device development      | It is desirable to de-identify and control by agreement or enclave public sharing model, and it is desirable to ensure the integrity and authenticity of the data      |
| Level 3      |   Business requirements: the person to be served is personally identifiable, but the surrounding people not easily identify him/her. Data content: Part of the personally identifiable information or code, separated from other information content, such as Zhang ××, queue number, etc. Data users: small range of people    | Notify the person to be served 	      |  Notify service recipients on public occasions, such as outpatient call, examination call, medical examination service call, etc.     |   Personal information needs to be partially masked, and the environment and the number of recipients are limited    |
| Level 4      |   Business requirements: must accurately identify individuals. Data content: contains complete and accurate personal health data. Data users: relatively small range of personnel, audit and privacy obligations    |   Personalized service and management	    |  Must accurately identify individuals, such as medical services for individuals, healthcare services, infectious disease control, genome sequencing, etc. Examples of scenarios: hospital interconnection, telemedicine, health sensor data management, mobile applications, commercial insurance integration     |  As it involves personally identifiable information, strict control between the environment and the recipient is desirable, and high standards are desirable to ensure data integrity and availability     |
| Level 5      |  Business requirements: necessary for the treatment of special diseases. Data content: Detailed information on special diseases. Data users: a very small range of personnel, audited, with confidentiality obligations     |   Medical service on special diseases    |  The diseases are sensitive such as HIV etc.	     | Strict identity identification, access control and other measures      |



### 8.2	By-Scenario Key Safety Measures
Based on the different scenarios of data circulation and use, the security aspects and responsibilities involved in the use of health data are different for each role, which determines the security control requirements to be met by each role. The security responsibilities and security measures that each role should take in different application scenarios shown in Table 4, and the security measures that need to be focused on for common scenarios are detailed in Chapter 11.

<center>Table 4. Highlights of data use security responsibilities and safety measures</center>

|Scenarios       | Security Point      |Security responsibilities and safety measures    |Examples       |
| --- | --- | --- | --- |
| Subjects -> Controller data flow      |<table><tbody><tr><td>Collection Safe</td></tr><tr><td>Transmission Safe</td></tr><tr><td>Storage Safe</td></tr></tbody></table> |<table><tbody><tr><td>Controllers: Informed consent for data collection</td></tr><tr><td>Controllers: Encryption, storage media control</td></tr><tr><td>Controllers: Domestic storage, encryption, classification and grading, de-identification, backup recovery, storage media control</td></tr></tbody></table>      |Examples of scenarios: doctors access medical data, health sensing data, mobile applications. Subject: Individual. Controllers: Medical institutions, research institutions, health insurance institutions, commercial insurance companies, health service companies       |
| Controller->subjects data flow.       | <table><tbody><tr><td>Transmission Safe</td></tr><tr><td>Use Safe</td></tr></tbody></table>      |  <table><tbody><tr><td>Controllers: Encryption, storage media control</td></tr><tr><td>Controllers: identity identification, access control, sensitive data control</td></tr></tbody></table>      | Scenario example: Patient query. Subject: Individual. Controller: Medical institution      | 
|Controller<->controller intra-data flow. | <table><tbody><tr><td>Collection Safe</td></tr><tr><td>Process Safe</td></tr><tr><td>Use Safe</td></tr><tr><td>Storage Safe</td></tr></tbody></table>       |<table><tbody><tr><td>Controllers: informed consent for data collection, approval</td></tr><tr><td>Processors: de-identification, permission management, quality management, metadata management</td></tr><tr><td>Controllers: approval of authorization, identity authentication, access control, auditing</td></tr><tr><td>Controllers: domestic storage, encryption, classification and grading, de-identification, backup recovery, storage media control	</td></tr></tbody></table>    | Example of scenario: Internal data usage. Controllers: Medical institution      |
| Controller<->processor data exchange.       |<table><tbody><tr><td>Transmission safe</td></tr><tr><td>Process Safe</td></tr><tr><td>Storage Safe</td></tr></tbody></table>       |<table><tbody><tr><td>Controller: Before transmission review, evaluation, authorization; encryption, audit, traffic control, storage media control.Processors: data transmission encryption, transmission protocol control</td></tr><tr><td>Processors: de-identification, permission management, quality management, metadata management, auditing.	</td></tr><tr><td>Controller: Domestic storage, encryption, classification and grading, de-identification, backup recovery, storage media control, manage the process of how processors storage data. Processors: Domestic storage, encryption, classification and grading, de-identification, backup recovery, storage media control, deletion mechanism.</td></tr></tbody></table>        |  Example of scenario: Medical device maintenance. Controllers: Medical institutions, government agencies. Processors: research institutions, health information service providers, medical device manufacturers.     |
|Inter-controller data exchange.       |<table><tbody><tr><td>Transmission Safe</td></tr><tr><td>Use Safe</td></tr><tr><td>Storage Safe</td></tr></tbody></table>       |<table><tbody><tr><td>Controller A: Integration security, encryption, auditing, traffic control, storage media control. Controller B: Integration security, encryption, auditing, traffic control, storage media control</td></tr><tr><td>Controller A: Approval of authorization, authentication, access control, auditing. Controller B: Approval of authorization, identification, access control, auditing</td></tr><tr><td>Controller A: Domestic storage, encryption, classification and grading, de-identification, backup recovery, storage media control, deletion mechanism. Controller B: Domestic storage, encryption, classification and grading, de-identification, backup recovery, storage media control, deletion mechanism	</td></tr></tbody></table>        | Examples of scenarios: Interoperation; telemedicine.Controllers: government agencies, medical institutions, health insurance agencies      |
|Controllers -> users data flow.       |<table><tbody><tr><td>Transmission Safe</td></tr><tr><td>Use Safe</td></tr><tr><td>Storage Safe</td></tr></tbody></table>        |<table><tbody><tr><td>Controller: Before transmission review, evaluation, authorization; encryption, audit, traffic control, storage media control</td></tr><tr><td>Users: approval of authorization, identity authentication, access control, auditing	</td></tr><tr><td>Controller: Domestic storage, encryption, classification and grading, de-identification, backup recovery, storage media control, manage the process of how users storage data. User: Domestic storage, encryption, classification and grading, de-identification, backup recovery, storage media control, deletion mechanism	</td></tr></tbody></table>        |Examples of scenarios: commercial insurance interface, clinical research, reuse. Controller: Medical institutions. Users: Commercial insurance companies, research institutions.       |

Note: In the actual application scenario of data, there is a controller corresponding to multiple use scenarios, when it is necessary to implement security measures with reference to multiple data exchange scenarios.

### 8.3  Key Safety Measures for Public Open
Different forms of data openness are applicable to: 
a) The "principle of least necessary" is followed.
b) The purpose, content, and users of the open data are approved by the Data Security Committee to ensure compliance with the requirements of legality, legitimacy, and necessity.
c) To the extent possible, de-identify the data according to the purpose of use.
d) Clarify the purpose of data development and use, the security responsibilities to be borne by the user, the security measures, etc., and sign the corresponding agreement; it is appropriate to conduct security assessment in accordance with the regulations for those involving transfer from the countries, and to conduct assessment and approval in accordance with the regulations for those involving important data. In addition, the security measures that need to be satisfied in different forms of data opening are detailed in Table 5.

Table 5. Key points of security measures for different ways of data opening
|Ways of data opening       | Key security measures    |
| --- | --- |
| Public website     | It is advisable to have the data security committee approve the public data|	
|File Share |   - 1) It is desirable to use cryptographic techniques to safeguard data integrity and traceability.  - (2) it is desirable to audit the volume, content and generated time of documents. - (3) Data transmitted via mobile media should be encrypted or access-controlled by mobile media solutions|
|API interface |(1) it is desirable to use password, cryptography, biotechnology and other identification technologies to identify the access user. (2) it is desirable to use verification technology or cryptographic technology to ensure the integrity of data in the communication process, and to ensure the confidentiality of data in the transmission process through encryption and other means, and the selection of encryption technology is desirable to consider the application scenario, data scale, efficiency requirements and other aspects; (3) It is desirable to conduct log auditing of API calls, including but not limited to caller, call time, call interface name, call result, etc. (4) it is desirable to take Web security measures to prevent attacks such as SQL injection, XSS, burst password, etc. |
|Online query|(1) anonymously accessible data are approved by the Data Security Committee to ensure that no personal information, important data, etc. are involved. (2) it is desirable to use password, cryptography, biotechnology and other identification technologies to identify the querying user. (3) it is desirable to use verification technology or cryptographic technology to ensure the integrity of data in the communication process, and to ensure the confidentiality of data in the transmission process through encryption and other means, and the selection of encryption technology is appropriate to consider the application scenario, data scale, efficiency requirements, etc. (4) it is desirable to audit the data volume, number of queries and query time of queries and form exception reports. (5) it is desirable to monitor bulk query operations and alert in time when high-frequency queries are found. (6) it is desirable to take Web security measures to prevent attacks such as SQL injection, XSS, burst password, etc. |
|Data analysis Platform | 1) it is appropriate for the export of any analysis results to be approved by the data security committee. 2) it is appropriate to manage the access to the platform, including access permission and data use rights; 3) it is appropriate to have traceability and traceability functions for data operations. 4) The exported data or results should be kept for pending audit |


## ９ Guide for Security management
### 9.1 Summary
In order to achieve the security objectives described in Chapter 5, it is appropriate for controllers to conduct data classification and grading, scenario analysis in accordance with the requirements of GB/T 22080-2016 with reference to Chapter 6, analyze the risks faced by health data security, adopt corresponding security measures, and check the effects of the implemented measures for continuous improvement.
Controllers can establish data use management methods with reference to Appendix C, approve data applications with reference to Appendix D, sign data processing (use) agreements with processors (users) with reference to Appendix E, and conduct self-inspection with reference to Appendix F.

### 9.2 Organization
It is advisable to establish a sound organizational guarantee system, with an organizational structure that includes at least a health data security committee and a health data security work group, to ensure good health data security management and to form corresponding documentation, including but not limited to:
a) Establish a health data security committee (referred to as the committee) to take overall responsibility for health data security and discuss and decide on major health data security matters, the committee should
    - 1) include the top management of the organization and the heads of each business port, etc.
    - 2) cover information security, ethics, legal, statistical, audit, confidentiality and other related professionals
    - 3) Have the top person in charge of the organization as the chief member.
    - 4) Can rely on existing ethics committees, hospital councils, etc., without having to re-establish them.
    - 5) coordinating the allocation of human, material, financial and other resources necessary for health data security work, such as security administrators, security auditors, system administrators, etc. based on the principle of separation of authority
    - 6） Responsible for reviewing the health data security strategy, risk assessment plan, compliance assessment plan, risk disposal plan and emergency disposal plan.
    - 7) Responsible for reviewing data security-related regulations (e.g. data use approval process).
    - 8) Responsible for reviewing de-identification strategies and processes.
    - 9) Hold regular working meetings, recommended to be held at least once a month.

b) Establish a health data security work office and designate a person (e.g., data security officer) to be responsible for the day-to-day work of health data security to
    - 1) Responsible for implementing the decisions of the Health Data Security Committee and reporting to the Committee.
    - 2) Responsible for establishing, maintaining and updating the health data security policy, risk assessment program, compliance assessment program, risk disposal program and emergency disposal program.
    - 3） Responsible for establishing, maintaining and updating data security related rules and regulations.
    - 4) Responsible for developing, maintaining and updating data use approval processes, as well as de-identification strategies and processes.
    - 5) Sorting out business processes and the health information systems and data involved, and conducting security risk analysis and compliance analysis, and making recommendations for health data security work.
    - 6) Form and manage the metadata structure to form a data and system supply chain structure that is consistent with business processes. 
    - 7) Be responsible for data security education and training of personnel to ensure that relevant personnel have appropriate data security capabilities. 
    - 8) Conduct a comprehensive self-examination of health data security at least annually and make recommendations for rectification. 
    - 9) Audit the use of health data and adjust and improve security measures when appropriate.
    - 10) Monitor and warn the security status of health data, and adjust and improve security measures as appropriate.

### 9.3 Process
9.3.1 Planning
The main tasks of the planning phase are as follows, and each task should be documented accordingly.
a) Define the scope of the health data security work, determine the work objectives, and establish a work plan.
b) Establish a health data security strategy and communicate it to the entire organization.
c) Establish rules and regulations related to health data security and inform the whole organization.
d) Establish a health data security risk assessment program and a compliance assessment program.
e) Sort out health data-related operations and the systems and data involved.
f) Identify health data security risks and assess the impact.
g) Identify health data security compliance risk points and assess the impact.
h) Establish a risk disposal plan for the risks; for those involving data use disclosure, it is appropriate to dispose of them in accordance with Chapter 7 "Use and Disclosure Principles"; for those involving network and system security, it is appropriate to dispose of them in accordance with GB/T 22081-1016 and GB/T 22239-2019; for those involving basic security and data service security, it is appropriate to dispose of them in accordance with GB/T 35274-2017; for cloud computing security, it is appropriate to dispose in accordance with GB/T 31168.
i) Review and adopt the risk disposal plan.
j) Establish a data security emergency disposal plan.

9.3.2 Implement
The main tasks of the implementation phase are as follows, and each task should be documented accordingly.
a) All aspects of the health data use and disclosure process should strictly implement the established data security-related regulations, security policies and processes.
b) Implement a risk management program, including the implementation of selected security measures.
c) Prepare appropriate resources, including human, material, and financial resources, to support the security work.
d) Conduct necessary information security education and training.
e) Implement effective control over the information security work carried out and the resources invested in information security work.
f) Take effective response measures for information security incidents.

9.3.3 Check
The main tasks of the inspection phase are as follows, and it is appropriate to document each task accordingly.
(a) monitor the process of work related to health data security, such as the implementation of security measures process.
(b) Regularly review the effectiveness of the implementation of risk management programs, including assessing the acceptability of the residual risk after the implementation of the corresponding measures, etc.
(c) Periodically check whether the health data use disclosure complies with Chapter 7 "Use and Disclosure Principles". 
(d) Periodically check that technical security and de-identification work is performed in accordance with Chapter 10. 
(e) Integrate the inspection process into the organization's internal management.
(f) Self-inspection or third-party inspection, as appropriate.

9.3.4 Improve
The main tasks of the improvement phase are as follows, and each task should be documented accordingly
(a) Improve security measures in response to monitoring or inspection results, including taking preventive measures or adjusting the content of business activities that may affect the security of health data.
(b) Establish a corrective action plan and implement it according to the plan.

### 9.4 Emergency Disposal
The main work of emergency disposal is as follows, and each work is suitable to form corresponding documentary records.
a) Establish emergency plans, including the conditions for starting emergency plans, emergency handling process, system recovery process, incident reporting process, post-event education and training, etc. It is advisable to regularly evaluate and revise the network security emergency plan, and organize at least one emergency drill each year.
b) It is advisable to designate special data security emergency support teams and expert teams to ensure that security incidents are dealt with in a timely and effective manner.
c) It is desirable to develop a disaster recovery plan to ensure that the health information system can recover from a network security incident in a timely manner and establish a security incident traceability mechanism.
d) After the occurrence of data security incidents, it is appropriate to dispose of them in accordance with the emergency plan; after the completion of the disposal of the incident, timely written reports on the incident to the security protection work department in accordance with the regulations, the content of which should include at least: description of the incident, analysis of the causes and effects, disposal methods and other information.
e) It is desirable to carry out a comprehensive assessment based on the security issues identified in the detection and assessment, monitoring and early warning and disposal results, and if necessary, to carry out risk identification again and update the security policy.

## 10 Security Technology Guide
### 10.1 General Security Technology
Controllers are advisable to manage data security in accordance with GB/T 22081-2016, GB/T 22239-2019 and GB/T 35274-2017, etc.
a) It is appropriate to provide the necessary security protection for information systems and network facilities and cloud platforms that host health data.
b) It is appropriate to implement data security measures for various activities in the data life cycle, including data collection, data transmission, data storage, data processing, data exchange, data destruction, etc., in order to reduce security risks and ensure data security.
c) It is advisable to take necessary security measures for data platforms and applications around the characteristics of each phase of the system life cycle such as planning, development, deployment, operation and maintenance, to establish a secure data management infrastructure, reduce the security risks of data platform and application operation, and guarantee business continuity.
d) It is advisable to classify and grade the management of health data, formulate and implement reasonable strategies and processes, and limit the use and disclosure to a minimum.
e) It is advisable to implement security measures such as identity identification, access control, security audit, intrusion prevention, malicious code prevention, and media usage management.
f) It is appropriate to ensure data quality to meet business needs and implement security measures such as backup recovery and residual information protection.
g) It is advisable to use cryptographic technology to ensure the integrity, confidentiality and traceability of data in the process of collection, transmission and storage; if storage media is used for transmission, it is advisable to implement control over the media.
h) When storing personal biometric information, it is appropriate to adopt technical measures for processing before storage, such as storing only the summary of personal biometric information.
i) It is appropriate to use cryptographic technology in accordance with the relevant requirements of national cryptographic management.
j) It is appropriate to comply with the relevant general requirements of important data management, critical information infrastructure security management and other policies.

### 10.2 De-identification
It is appropriate for controllers to carry out de-identification in accordance with GB/T37964-2019, and de-identified data should be applied to controlled public sharing or enclave public sharing (an environment fully controlled by controllers), and it is appropriate to agree on the purpose, manner, duration, and security measures for data use through data use agreements. The de-identification strategy, process and results are desirable to be approved by the Data Security Committee. When data are applied to clinical research and pharmaceutical/medical R&D, the relevant requirements are as follows.

a) It is appropriate to remove information in personal attribute data that can be uniquely identified with an individual or whose disclosure would have a significant impact on the individual, such as: name; ID card/driver's license and other document numbers; telephone numbers, faxes, e-mails; health insurance numbers, medical record numbers, accounts; biometric information (information unrelated to the purpose of the application such as fingerprints, voice, etc.); photographs; hobbies, beliefs, etc.
b) Information in personal attribute data that can be indirectly associated with individuals is appropriate for generalization, conversion, and other processing, such as
    - 1) information such as employer, address, postal code, etc., if the employer information or the population covered by the combination with other information is more than 20,000 people, the employer information can be retained; if the address information includes province (municipality directly under the Central Government), city (county), street (township) or the population covered by the combination with other information is more than 20,000 people, it can be retained, otherwise it is appropriate to remove the street (township) to ensure that the population covered by the combination is above; if the population covered by zip code information or combined with other information is above 10,000, it can be retained, otherwise it is appropriate to set the low zip code to '0' to ensure that the population that can be covered is above 10,000.
    - 2) Generalization of a specific age, for example, giving an age range. For example, 38 years old can be converted into 30 to 40 years old to ensure that the number of people meeting the same age condition in the same region is more than 20,000.
    - 3) Birthdays and all other date information, e.g., admission time, discharge time, can only be specific to the year, or time drift processing.
c) It is appropriate to remove the names of health workers and other identifying information.
d) It is desirable to have a minimum number of 5 or more people in the dataset with the same value for all attributes.
e) For cases that need to be traced back to a patient, it is appropriate to create a patient code index within the controller.
f) The configuration of various parameters used in the de-identification process, such as time drift ranges, patient code indexes, and various individual code generation rules, should be kept strictly confidential and limited to the controller's internal dedicated management.
g) When re-identification is required to identify the subject, it is desirable that the process be handled by the controller's internal personnel and that the process be kept strictly confidential.
h) It is desirable to prohibit data users from participating in the work related to de-identification.
i) It is advisable to sign a data use agreement to govern the purpose and duration of data use and data protection measures, etc.
j) In a controlled public sharing model, it is appropriate for users to record data usage and be audited by the controller.
Related examples are shown in Table 6. The identifier categories of health information data elements and the proposed de-identification methods can be found in Appendix G.

Table 6. Examples of De-identification
|Attribute   |Suggestions for deidentify   |Applicable data   |
|---|---|---|
| Name  |Suggest to delete or set to empty    |  Subject's name, Doctor's name, Researcher's name, Family member's name|
| Contact	| Suggest to delete or set to empty or generalize. For example, the address should only be specified to the city or county level, hiding the address below the county level| Personal telephone number, email, account, address|
| Date	|It is suggested to adopt the "time shifting method", conversion method, or generalization. For example, different random offset values can be defined for different research projects, and data disturbance can be achieved by adding or subtracting the random offset value from the date and time, to achieve data de-identification. For example: Admission date 2018-01-01 + Random offset value 100 = Admission date: 2018-04-11, Discharge date 2018-04-01 + Random offset value 100 = Discharge date: 2018-07-10.  Discharge date - admission date = 90 days.  Through this method, data de-identification can be ensured while ensuring the correctness of the calculation logic. The conversion method is to replace the result obtained by using it and other dates with operations, such as the length of stay. Generalization only retains the year and month, or even only the year.	| Time information in medical application that can be associated with individuals through data analysis: for example, admission date, discharge date, surgery date, etc|
| Birth date	| It is suggested to delete, set to empty, or replace with age.	|Birth date|
|Age	| It is suggested to use the method of "data generalization." For example: --- Age ≤ 89 or > 89 --- Age interval <25, 25-29, 30-34, ..., 85-89, >89. Note: > 89 cannot be further sub-divided.| Age |
| Number	| It is suggested to delete or set to empty. If it is necessary to use the uniqueness of numbers for logical analysis, such as judging whether multiple medical records belong to the same person through ID numbers, randomization based on the original data can be used to generate unique identification for replacement. If it is necessary to use implicit geographical information such as postal codes, disturbance and generalization methods can be used for processing, such as: original postal code record 100080, de-identified as 100* * * after generalization.	| ID card number, social security card number, work permit number, and residency card number.|
|internal number assigned by medical institutions| It is recommended to replace or delete them. If these numbers are needed for logical analysis, a unique identifier can be generated based on the original data through randomization. If these numbers are not needed for logical analysis, they should be deleted.	| Exam result report number, check report number, hospitalization number, outpatient (emergency) number, etc.|

## 11 Data Security on Typical Scenarios(Skip)