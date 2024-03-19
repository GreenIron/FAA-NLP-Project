# A NLP project on FAA documents

## Abstract
This is an in-work personal NLP project dealing with FAA documentation: use case definition, data source identification, data collection, data preparation, data mining and some RAG and LLM fine-tuning.

# Introduction

## Motivation
While aviation is a heavy producer of documentation, most of the aviation-related database publicly available on popular ML websites (kaggle, huggingface, google data) is about unformatted tabular data (e.g. accident rates, airport traffic, helicopter types) and not about NLP. In the same time legal has had a several datasets available. This is quite unusual considering that aviation data is very good at producing complex, high-quality and standardized documents or more generally any sorts of communication (e.g. GAMA standards, ATC standards).

While it takes at least 10 years for a commercial-level technology to hit aviation, data has been at the core of airborne aviation for a lot quite some times (CVFDR, telemetry...). FAA has been offering for several years a modern and powerful platform to browse all the publicly available FAA documentation: [DRS](https://drs.faa.gov/). This platform offers tremendous capabilities and an API (one of the many DOT platforms) to programaticaly acccess its documentation.

I truely encourage you to take a look at it right now since this will be the primary source of data.

Non-governmental platforms like [AHST](https://ushst.org/) offer also high quality documentation (this one on helicopter safety), but have no API.

Most of the ACs and TSOs rely on industry standards published by SAE or RTCA. Those have been excluded since they are not publicly available (and they are not cheap).

NASA standards have also been excluded since they do not stritcly belong to the same usage domain (my focus is aviation and NASA standars are almost never recommended).

I did not scrap the FAA website since I anticipate that this would be rather redundant with [DRS](https://drs.faa.gov/). Some specific items like the [Rotorcraft Issues List](https://www.faa.gov/aircraft/air_cert/design_approvals/product_issues_lists/rotorcraft) are not on [DRS](https://drs.faa.gov/).

## Why aviation data
The starting block of any ML project is getting intimately familiar with the data. Since I work in aviation (rotorcraft), I've a professional interest in knowing more about all types of documents (format and content) that the FAA produces (with a focus on rotorcrafts).

Data collection and mining is systematically overlooked in most of the ML projects ([Conway's law](<https://en.wikipedia.org/wiki/Conway%27s_law)>)?), leading to post-training issues. So starting at data collection level is an interesting exercise for airborne application.

From a pure ML perspective, aviation data offers lots of benefits:
* because of the safety constraints, aviation structuraly produces lots of data in various format 
* aviation data is diversifed (e.g. produced and consumed by people with various background) and standardized (FAA Forms\, Tech logs, ATC communication, ARINC 429 GAMA, accident reports...)
* aviation data has been around for many decades

And more specifically for the text documents released by the FAA:
* FAA offers a powerful API
* FAA documents are well formatted (Q&A tables, consistent tables of content)
* FAA documents have a high quality standard (Order 1000.36) compared to the best NLP dataset that one can find on Hugging Face. The notably natively include ["chain of thoughts"](https://arxiv.org/pdf/2201.11903.pdf)-like answers.

Overall I ended with the following datasets ([parquet](https://parquet.apache.org/) format):
* STCs with metadata, extracted description and limitations/conditions (~50MB)
* DRS with metadata (documents requiring OCR have no text) (~100MB)
* ACs, Orders with metadata and splitted paragaraphs (>1GB)

# Data Collection
## Results on data collection
DOT APIs are very easy to use and these are the steps that I followed to collect DRS data:
1. Get familiar with the API: https://drs.faa.gov/help/helpdetails
2. Download the DRS Document Types Metadata Mapping file from https://drs.faa.gov/help/helpdetails
3. Collect the index table for each DRS doc type (~ 1 hour)
4. Collect each file or text from each document found in each index table (several months)

Check [#10](./10_stc_collection.ipynb) for STCs and [#90](./90_gendoc_collection.ipynb) for all DRS.

This is pretty straightforward, but I did hit several hard points:
1. DRS website changed a little bit, so the code had to follow
2. Some files caused some issue (filename with special characters like &)
3. After some times my IP address got banned, so I started using a VPN to finish my data collection
4. On top of the main file, DRS offers also "Attachments/Public Comments" documents. As of today those are not searchable with DRS and not in the DRS Document Types Metadata Mapping file.
5. There also non-pdf and non-text files

Aeronautical Information Manual (AIM) and Aeronautical Information Publication (AIP) are two basic references in aviation. They are not in DRS, so I've added them manually to the dataset.

[eCFR](https://www.ecfr.gov) is also another great ressource (can collect dockets from there), but it is already covered in [DRS](https://drs.faa.gov/).

## Other sources of data worth exploring
The following data could also be an excellent source of data for an NLP use case, but I've not had the time to focus too much on those:
* public comments are organized as Q&A tables. They are generally very nicelly formatted so that a tabular extrator library should work (and even sometimes directly in csv format). The answer provided by the FAA is not "right" like what can be found in other factual Q&A database, but this could still be a good starting point.
* AC 43-16A - General Aviation Maintenance Alerts offers also an excelent source of maintenance database
* MMELs are also well formatted, but definitely less diversified
* FAA DER training center (learning documents and Q&A)
* all the DRS missing data and all the documents which were discontinued to exist. Identifying and explaining missing data is important for data quality and bias mitigation.

# A first try with STC (Supplemental Type Certificate)

## Why STC certificates
STC is the master document that defines the type design of an aircract modification, it's the final document (except for the PMA) that gets issued at the end of a certification project. I happen to be working a lot with STCs, so I knew that those enjoyed lots of nice properties:
* small (<1000 tokens so local LLM can be fine-tuned)
* rigid structure
* informative
* diverse
* long history 

## Metadata only (with a focus on rotorcrafts) [#20](./20_stc_metadata_exploration.ipynb)
The DRS metadadata already offers a wealth of information before even exploring the text content. I've only analysed a fraction of the metadata and the following is another fraction of this analysis. It's really a high-level analysis to get to know the data better, but I didn't push towards more complex data mining methods (dimensiality reduction, clustering... scikit-learn is the best source I know to start a ML project). Maybe something for later.

The metadata file has as many rows for an STC as there are issuances. Date of STC (re)-issuance is given by "stcStatusDate". A STC is reissued for example to add a new configuration (e.g. obsolescence). Note that a STC can be updated (e.g. minor change for editorial changed) and not reissued and in that case the latest update will be indicated in the field "docLastModifiedDate".
"drs:stcAplicationDate" corresponds to the application date for the firs STC issuance.

* Number of STC (documents):
  * indexed: 76416
  * with PDF (usable for text content analysis): 43818

* Sanity check:
  * only one broken PDF ([SA00765DE](https://drs.faa.gov/browse/excelExternalWindow/2F49BD200CEC24DD86257F5600610445.0001?modalOpened=true))
  * very few abnormal dates (<50)
  * very few headers typos for the DRS metadadata ("Document Type name in DRS ", "Deafult  Sort By")

* *Rotorcraft STCs reissuance distribution*: there is a period (which is remarkable) that is ~3 years, but looking more closely reveals that the actual period is more like 1.6 years (there is another peak around 7 years (~1.7*4)). It could be interesting to filter by ATA, company, ACO... to try to consolidate or explain this pattern. Another other interesting observation is that is period is not consitent when we switch to airplanes.

![plot](./images/20_rotorcraft_reissuance.png)

* *Number of STC re-issuance for rotorcrafts*: the majority of STCs are not reissued (but there could have been some minor changes to the type design).

![plot](./images/20_number_of_STC_issuances_for_rotorcraft.png)

* *Number of current and historical STCs for the different FAA offices (all years)*: it's quite expected to see Fort Worth on top of the list for rotorcrafts.

![plot](./images/20_Number_of_STCs_by_Office_all_years.png)

* *Current STCs re-issuance for the different FAA offices since 2020*: interrestingly Fort Worth is not on the top position while the group of other top FAA offices remains the same.

![plot](./images/20_Number_of_STCs_by_Office_2020_rotorcraft.png)

* *Year of initial applications*: I was surprised to observe that new rotorcraft STCs have only been decreasing since 2008 (US financial crisis, but it might be just a correlation). We can also see a sharp drop caused by COVID.

![plot](./images/20_rotorcraft_stcApplicationDate_by_year.png)

* *Year of STC reissuance*: we can still see the decrease, but it's unclear what caused those 3 peaks without drilling more into the data (probably an artifact).

![plot](./images/20_rotorcraft_stcStatusDate_by_year.png)

* *Application day (of the week)*: I would have expected a flat distribution, but it's interresting to see a peak at the beginning/end of the month. This might be driven by company business objectives. Harder to explain is the peak in the middle of the month.

![plot](./images/20_rotorcraft_stcStatusDate_by_day.png)

* *Application day (of the month)*: The distribution is rather flat as expected, with a sharp drop on Friday (not surprising). Most surprising is that there are a few applications on Saturdays.

![plot](./images/20_rotorcraft_stcStatusDate_by_day_of_week.png)

* *Helicopter STCs by holder between 1990 and 2000*: a big surprise to me was that American Eurocopter (now Airbus Helicopters, Inc.) was one of the largest issuers of STC certificates (but a detailed analysis of the STCs show that those are not complex) while still reamaining a rather small company. It's also worth noting that companies have very different business models (ODA doing work for third parties, ODA doing only internal work, companies with core products working with ACO).

![plot](./images/20_STC_by_STC_holder_helicopter_1900.png)

* *Historical and Current STCs*: the distribution is quite expected, but most surprising is that the distribution is not reflecting the relative weight of those markets (large multi-engine should be significantly highier).

![plot](./images/20_STCs_by_stcProductSubType.png)

I could keep going on and on (e.g. looking at time drifts, comparing single engine vs. multi-engines, focusing on a region or on a company, crossing with other database...), but that's enough histograms for today.

## Text analysis
### Getting to know the data
STC textual data was decoded using a mix of from pypdf and PyPDF2 for pure PDF decoding and Google Cloud Platform OCR when only a scan was available.
GCP OCR proved to quite uneffective on a fraction of scanned pdfs (SA00005MC-D) yielding very odd artifacts (unable to decode some letters, paragraphs in the wrong order). GCP OCR does not offer any options to adjust performances, so the only workaround was to perform some image preprocessing (was able to improve performance).

STCs are well defined and the format has remained pretty much the same over the years. The key fields from a text standpoint are the "Descriptions" and the "Limitations/Conditions". The rest of the data is quite redundant with the metadata. Let's look at some examples first:
* accross the ages: [old](https://drs.faa.gov/browse/excelExternalWindow/BA172489A7E9DA1586257CD30047CF76.0001?modalOpened=true) , [recent](https://drs.faa.gov/browse/excelExternalWindow/DRSDOCID128075530320230314172310.0001)
* description: [short](https://drs.faa.gov/browse/excelExternalWindow/CA21490D878D3C63862574C600496658.0001?modalOpened=true), [average](https://drs.faa.gov/browse/excelExternalWindow/3FE1351336C0748F8625831400585C06.0001?modalOpened=true), [long](https://drs.faa.gov/browse/excelExternalWindow/17914C2EEDFFC8C98625804300770386.0001?modalOpened=true)
* limitations/conditions: [short](https://drs.faa.gov/browse/excelExternalWindow/DRSDOCID125280334620221128154006.0001?modalOpened=true), [average](https://drs.faa.gov/browse/excelExternalWindow/3FE1351336C0748F8625831400585C06.0001?modalOpened=true), [long](https://drs.faa.gov/browse/excelExternalWindow/EF198F621A3B2AA9862585D90044ACBF.0001?modalOpened=true)

The following two pictures show the word distributions for those two fields. Description is a Gama distribution, while Limitations/Conditions is more multimodal. That's not surprising since the Description is more formatted, while the Limitations/Conditions field is often used a field to register auxiliary information.

![plot](./images/22_number_of_words_in_description.png)

![plot](./images/22_number_of_words_in_limitations.png)

### (Lack of) Results [#22](./22_stc_content_exploration.ipynb) [#25](./25_stc_text_exploration.ipynb)
I failed to find a decent LLM application:
* zero shot using 7B LLMs (LM Studio) yielded poor results (classification: guessing meta data based on Description, Limitations/Conditions, guessing the ATA).
* ATA-classification (using DRS MMEL and JASC) performances were poor.
* Overall I could not find any interresting use cases (using [nlp book](https://www.oreilly.com/library/view/natural-language-processing/9781098136789/) as a basis).

The key problems might be that the STC Description and Limitations/Conditions are not informative enough and are quite independent between each others from the metadata (there are no patterns to learn or represent). Another problem is that the Description is often based on the commercial name of the modification/equipment, which is too much of a specific usage domain. Unlike a catalogue, it is not meant to provide a commercial description. We would need to scrap data from all STC holders, correlate the P/N with the catalogue description to have more information.

Following picture shows the lack of correlation between Description and a Limitations/Conditions. It's worth noting that there are some obvious vertical and horizontal bars. Those might be linked to "standard sentences" ([AC No: 20-188](https://www.faa.gov/documentlibrary/media/advisory_circular/ac_20-188.pdf)). There are also several cluster that could be worth exploring (same underlying pattern or pure coincidence?).

![plot](./images/22_limitations_vs_descriptions.png)

Other than that my main findings are that:
* There is some overlap between what's a Description vs. a Limitations/Conditions. For example the RFMS and ICA can be found in one on the other.
* Limitations/Conditions is sometimes used for additional configuration control.
* There is a wide variability on how those fields are used (depth of explainations).

# All DRS and the other non DRS documents

## Data collection [#90](./90_gendoc_collection.ipynb)
It took me about 3 months to collect all DRS data on my personal computer (working 1 or 2 hours before going to bed, so not a full time job), but many more months to set up a functional data collection routine. I collected all the DRS data, plus a few more (AIM, AIP). There was no signigicant difficulties, except being patient and improving the code to come up with any exceptions found on DRS (e.g. new data format, attachments, time-out, pdf crashes). 
The whole raw documents dataset (mainly pdf, but also doc, xls, html) is about 55GB.

## Getting to know the data
At the time of my data collection, there was:
* ~172799 indexed documents
* ~324805 available documents if we add the attachments. Those include a lot of redundant documents, because DRS makes available doc and html versions of some pdfs.
* ~171254 documents that can be decoded without OCR (I didn't try to OCR all the missing documents for budget reasons). Non-decodable OCR also include documents with no content (only the index is available).

This dataset is currently qui airplanes/rotorcrafts-oriented, but the FAA is adding more and more data for space travels.

The following pdf made PdfReader crash really bad, but I didn't investigate further (I just noted that those happened to be the oldest indexed FAA documents):
* [CAM 1: Supplement No. 1; New Issuance System - Change of date of CAM 1 to December 15, 1959](https://drs.faa.gov/browse/excelExternalWindow/FAA000000000000000CAM1_121559PDF.0001?modalOpened=true)
* [CAM 4b: Supplement No. 1; Certification and Operation of Certain Airplanes forthe Department of the Interior in the Trust Territory of the Pacific Islands](https://drs.faa.gov/browse/excelExternalWindow/FAA00000000000000CAM4b_050160PDF.0001?modalOpened=true)
* [CAM 51: Supplement No. 2; 51.17 - Standard of Performance](https://drs.faa.gov/browse/excelExternalWindow/FAA00000000000000CAM51_060851PDF.0001?modalOpened=true)

I skipped the metadata analysis and focused only a the tokens to have an idea of what NLP tool to use.

The following table recaps the number of documents by DRS document type:
DRS doc types|ADFRAWD|ADNPRM|AC|AB|POLICY|CAM|CAR|CANIC|ADFREAD|ELOS|EXEMPTION|CFRFRSFAR|NORSEE|NPRM|PMA|SAIB|SCFINAL|SCPROPOSED|SFAR|STC|TSOI|TSO|FAR|TCDSMODEL|UNAPPROVED_PARTS_NOTIFICATIONS|ENVIRONMENTAL_SUPPORTING_DOCUMENTS|LAUNCH_SITE_OPERATOR_LICENSES|LAUNCH_VEHICLE_OPERATOR_LICENSES|PERMITS|REENTRY_SITE_OPERATOR_LICENSES|REENTRY_VEHICLE_OPERATOR_LICENSES|SAFETY_ELEMENT_APPROVALS|ORDER_8900.1|CLARIFY_POLICY|AFS-1_MEMORANDUMS|AIRCRAFT_MASTER_SCHEDULE|AIRCRAFT_STANDARDIZED_CURRICULUM|AT_JTA|OTHER_AWO|ALERTS|OTHER_AWARDS_INFORMATION_GUIDES|8900.1_EDITORIAL_CORRECTIONS|OTHER_EFB_RESEARCH_REPORTS|OTHER_EFB_CHECKLISTS|OTHER_FAA_90_DAY_SAFETY_REVIEW|OTHER_CPD_6.03|AFS_FFS_UPDATEPUB|AFS_FFS_UPDATES|FOEB|FSB_REPORTS|AFS_POLICY_DEV_MEMOS|OTHER_FLIGHT_STANDARDS_ORM_WORKSHEETS|AFS_FOCUS_TEAMS|GA_JTA|BULLETINS|OTHER_PS_HANDBOOKS|INFO|OTHER_INFORMATION_GUIDES|PILOT_QUALIFICATION_CURRICULUM|OTHER_INTERNATIONAL_PUBLICATIONS|OTHER_JOB_AIDS|OTHER_LASER_INVESTIGATION_REFERENCES|MMEL_POLICY_LETTERS|MMEL|NOTICES|OSR|OPSS_GUIDANCE|OSWG|ORDER_8300.10|ORDER_8400.10|ORDER_8700.1|ORDER_8740.1|ORDERS|PART_129_OPSPEC_JA|OTHER_PS_FEDERAL_AVIATION_ACTS|OTHER_PS_LEGAL_INTERPRETATIONS|OTHER_PS_POLICY_MEMORANDA|OTHER_PS_PREAMBLES|OTHER_QMS_AND_BP|OTHER_RCCB|OTHER_RIRTP|SAFO|SAS_AXH_DCT|SAS_DCT|STCRELIEFAPPLETTER|8900.1_SUMMARY_OF_CHANGES|OTHER_SPRS|OTHER_VPM|LEGAL_INTERPRETATIONS
---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---
Number of documents|46666|15168|4212|20|1489|992|4966|0|730|3888|57658|4745|30|1035|56999|2682|2893|736|176|0|7137|1186|24255|15389|383|72|30|76|22|0|0|14|16832|0|0|3|3|2244|0|429|56|238|12|4|2|0|23|0|501|1377|0|0|29|1801|8643|588|499|29|3|40|17|0|872|10397|5133|94|58|490|1045|1311|653|181|4478|48|172|0|0|0|0|18|0|388|1073|5172|683|3441|0|0|2076

I wanted to analyse the documentary inflation over the years, but I stopped since it would be wrongly interpreted. 

* *Word distribution for all DRS*: As expected the distribuytion is multimodal with a heavy tail. It's interresting to note that the average size is rather small and could be entirely fed to a local LLM (~1k). This limitation does not apply to GAFAM provided LLM (100k to 1M tokens).

![plot](./images/27_token_distr_all_DRS.png)

* *Word distribution for ADFINAL*: this one present a very odd distribution with its two bell-shaped multimodal. It woulbe be interresting to understand if those two modes correspond to two different patterns.

![plot](./images/27_token_distr_adfinal.png)

* *Word distribution for AC*: I was more surprised to see that the AC follows a classic exponential decrease, because I'm used to very very long ACs (AC27 and AC20).

![plot](./images/27_token_distr_ac.png)

* *Word distribution for MMEL*: I was expecting a distribution concentrated around a single mode, but the MMEL distribution is more complex.

![plot](./images/27_token_distr_mmel.png)

* *Word distribution for ORDERS*: Same comment as AC.

![plot](./images/27_token_distr_orders.png)

## ML applications
The size of the dataset makes the project intractable on my personal computer, so I decided to narrow-down the usage domain.

As far as I know there are several ways to work on a document with LLM (arxiv source here): pass the document corpus in the context window (only available on cloud-based solutions - extremely resource intensive during training and inference), LLM fine-tuning (corpus is virtually stored in the weights - very resource intensive during training), RAG (corpus is stored in a flexible database - manageable with a personal computer).

[Retrieval Augmentation Reduces Hallucination in Conversation](https://arxiv.org/abs/2104.07567)

I use the following prompt:
* *role*: you are an expert in all aspects of operations, airworthiness and certification for rotorcraft and airplanes including maintenance and engineering.
* *task*: you provide detailed guidance to an applicant or an operator.
* *context*: rotorcraft and airplanes operations and airworthiness in United States National Airspace. You are using information from the following types of document: Aeronautical Information Publication (AIP), Aeronautical Information Manual (AIM), Advisory Circulars (AC), FAA Orders, FAA Order Handbooks, Federal Aviation Regulations (Title 14 CFR FAR).

### Working with a subset: AC, ORDER, HANDBOOK, AIP, AIM, FAR
The DRS database is made of various document types that do not belong to the same usage domain (e.g. space travels vs. civil flights). Some documents are also a mere database (e.f. PMA).

I decided to focus on a subset of core-engineering-aviation-knowledge documents. Those documents are also generally well written and contain detailed answers (important for LLM performance). The main issue is that I had to develop a manual tool to split those documents into sub-sections (also very important for LLM performance). This took me a while using regex, see [#92](./92_split_paragraphs.ipynb). Many of those documents (e.g. AC 29 which is about 1000 pages) deal with many topics.
* [Advisory Circulars](https://www.faa.gov/regulations_policies/advisory_circulars/) provide guidance to operators, maintainers, installers, applicants...
* [Orders](https://www.faa.gov/regulations_policies/orders_notices/) detail procedures that the FAA follows.
* [Handbooks](https://www.faa.gov/regulations_policies/handbooks_manuals) explain basic aeronautics knowledge.
* [Aeronautical Information Publication](https://www.faa.gov/air_traffic/publications/atpubs/aip_html/part1_gen_section_0.1.html) is basic airman knowledge.
* [Aeronautical Information Manual](https://www.faa.gov/air_traffic/publications/atpubs/aim_html/): is more basic airman knowledge.
* [Fereral Regulation](https://www.ecfr.gov/current/title-14): the US federal regulation.

I could have used other documents (TSO)... but that's already a good start.

We end up with the majority of documents below 5000 tokens (see histogram below), which is high but still manageable with RAGs with sparse encoding. We also have many documents below ~ 300 words which is good news also for performance (dense encoding). It is also obvious that there were several issues that led me with documents with an unacceptable number of words. I finally want to stress out that this dataset includes also historical (obsolete) documents.

![plot](./images/94_number_of_words_ACsORDERS.png)

#### RAG

##### Why RAG
Short intro based on https://haystack.deepset.ai/ and [nlp book](https://www.oreilly.com/library/view/natural-language-processing/9781098136789/).

Explain why I chose RAG (much better at not hallucinating),

Explain why I chose [haystack.deepset.ai](https://haystack.deepset.ai/) over other ones ([SciPhi-AI](https://github.com/SciPhi-AI/R2R), [langchain](https://www.langchain.com/), [LlamaIndex](https://www.llamaindex.ai/), [Pinecone](https://www.pinecone.io/), [Weaciate](https://weaviate.io/)).

#### LLM fine tuning
Expand on fine tuning based on [this](https://generallyintelligent.substack.com/p/fine-tuning-mistral-7b-on-magic-the), [this](https://helixml.substack.com/p/how-we-got-fine-tuning-mistral-7b) or [this](https://www.oreilly.com/library/view/hands-on-large-language/9781098150952/) and [this](https://www.answer.ai/posts/2024-03-06-fsdp-qlora.html).


#### Gemini 1.5

##### Zero-shot with no context window
Using the promp.

##### Zero-shot with context window
Gemini 1.5 accepts windows of 1,048,576 tokens, which makes it possible to upload AC 27-1B (~1000 pages) and a few smaller ones.


##### Fine-tuning with context window
It is also possible to fine tune Gemini 1.5 via Google Vertex inteface.

#### Results



# [ARP6983](https://www.sae.org/standards/content/arp6983/) Criteria Review

## Minimum information to present to the regulator
Based on FAA presentations and AC 20-166A.

### Introduction
Provide advisories related to general airworhiness aviation.

### Data
Data is real and distribution have been explored before. No data split here (~surrogate model). Data source is US GOV.

### Data management
Data stored as parquet files locally. With Haystack's RAG, it is using their [DocumentStore](https://docs.haystack.deepset.ai/docs/document_store).

### System/Model Architecture
Haystack's RAG.

### Performance
Only manual tests for now.

### Tolerance and Performance Requirements
No expectations.

### Additional Justification
RAG performs better than pure LLM for hallucination. Overall it is probably unacceptable since the HMI is the worse that it can be (pure texts).

## MLDL objectives

### 1 ML Constituent ODD
* *1.1 The MLCODD is characterized*: Yes. The ODD is actually explicitly defined in the dataset itself for most of it. Otherwise it is detailed on the FAA website.
* *1.2 AN ML Data impact analysis is developed from the MLCODD*: Unclear?
* *1.3 Any limitation of use to the ML-based System/Subsystem processes is provided, including the System Safety Assessment process*: None.

### 2 ML Constituent Requirements
* *2.1 ML Constituent requirements are developed for each ML Constituent*: 
* *2.2 Derived ML Constituent requirements are defined and provided to the ML-based System/Subsystem processes, including the System Safety Assessment process*:

### 3 ML Environment Set Up
* *3.1 The MLDL environment is selected and defined*: Based on conda [`environment.yml`](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#activating-an-environment).

### 4 ML Data Management
* *4.1 Data sources are identified and selected*: Yes (DRS).
* *4.1 Lack of undesirable bias is ensured*: Should be N/A.
* *4.3 Data required for training, validation, and test datasets is collected*: Yes (DRS).
* *4.4 Data has specified characteristics (such as representativeness, sufficiency, fairness, etc.)*: Yes (DRS).
* *4.5 Data are cleaned up and enhanced and can be used by the ML algorithm without any additional checking*: Yes (basic cleaning). One difficulty is that the raw data (pdf) are not usable as is and need to be decoded.
* *4.6 Requirements allocated to the ML Data are satisfied*:
* *4.7 Data pre-processing description is updated*: Yes.
* *4.8 Data leakage is avoided*: Yes (ensured by data source)*: Yes.
* *4.9 Input dataset is divided into three datasets for training, validation, and test*:

### 5 ML Model Design Model
* *5.1 The ML Model architecture is developed from the ML Constituent requirements*:
* *5.2 ML Model requirements are developed from the ML Constituent requirements and the ML  environment*:
* *5.3 Derived ML Model requirements and their rationale are defined and provided to the ML Constituent requirements process*:
* *5.4 The ML Model is built, trained, and optimized from the ML Constituent requirements and if applicable from the ML Model requirements*:
* *5.5 ML Model Description is developed from the ML Model to describe its architecture, its hyperparameters and parameters, its analytical/algorithmic syntax and semantic, its replication criteria, and its execution environment*:

### 6 ML Validation
* *6.1 ML Model requirements comply with system/subsystem requirements*: No (no specification).
* *6.2 ML Model requirements are accurate and consistent*: No (no specification).
* *6.3 ML Model requirements are compatible with the target computer*: No (no specification).
* *6.4 Data processing description is compatible with the target computer*: No (no specification).
* *6.5 ML Model requirements are verifiable*: No (no specification).
* *6.6 ML Model requirements conform to standards*: No (no specification).
* *6.7 ML Model requirements are traceable to system/subsystem requirements*: No (no specification).
* *6.8 ML Model Architecture Verification*: I don't understand.
* *6.9 The ML architecture is compatible with the ML Model requirements*: No (no specification).
* *6.10 The ML architecture is consistent*: I don't understand.
* *6.11 The ML architecture is verifiable*: I don't understand.
* *6.12 The ML architecture conforms to standards*: Yes, RAG architecture.

### 7 ML Model Verification
* *7.1 The ML Model complies with the ML Model requirements*: [impossibble since Hallucination is Inevitable: An Innate Limitation of Large Language Models](https://arxiv.org/abs/2401.11817)
* *7.2 The ML Model is accurate and consistent*:
* *7.3 The ML Model is compatible with target computer*: No target computer.
* *7.4 The ML Model is verifiable*: It's minimized, see [Retrieval Augmentation Reduces Hallucination in Conversation](https://arxiv.org/abs/2104.07567) and 
[Chain-of-Thought Prompting Elicits Reasoning in Large Language Models](https://arxiv.org/abs/2201.11903).
* *7.5 The ML Model conforms to standards*: Yes (reuse of state-of-the-art RAG).
* *7.6 Verification of verification procedures*: No. But [Automated Unit Test Improvement using Large Language Models at Meta](https://arxiv.org/abs/2402.09171) 
* *7.7 Test procedures are correct*: No.
* *7.8 Test results are correct, and discrepancies explained*: No.
* *7.9 Test coverage of ML Data and Model requirements is achieved*:  No.
* *7.10 Test coverage of ML Model structure to the appropriate coverage criteria is achieved*: No.

## Data quality learning assurance objectives
* *1 Nature of data (explicit definition of input variables)*: Textual with metadata (dates and categorical)
* *2 Ranges of data (minimum and maximum value, classes of categorical data)*: Can be achieved with a [`describe`](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.describe.html) or [`unique`](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.Index.unique.html#pandas.Index.unique).
* *3 Representativeness: distribution of data across MLCODD [numerical values distribution, flight or operating condition, ambient condition indicator (e.g., day temperature for a pressure sensor, day, or night indicator for an image, rain, or dust level, etc.)]*: by design the data is rather exhaustive, but missing other commercial standards that are FAA accepted (SAE, ASTM, MIL-SPEC, RTCA).
* *4 Fairness (lack of bias)*: Yes, based on FAA objectives (by data source).
* *5 Acceptable data sources and Source tag*: Yes (US Gov).
* *6 Accuracy*: Yes (by data source).
* *7 Resolution (Precision)*: Not applicable.
* *8 Sufficiency: Minimum acceptable size for datasets*: Yes for anything that does not require an issue paper.
* *9 Criteria for data cleansing and denoising (missing data, out-of-distribution data)*: Unclear.
* *10 Criteria for data transformation*: N/A.
* *11 Criteria for data augmentation, if applicable*: For NLP, data aurgmentation relies on the use of synomics, back-translations, regeneration...
* *12 Criteria for data annotation, if applicable*: N/A.
* *13 Criteria for data segregation (training, validation, and verification datasets)*: Not different from standard practives.
* *14 Assurance Level*: N/A (DAL E).
* *15 Traceability to system level*: Not compliant.
* *16 Timeliness (Time stamp)*: FAA regularly cancels its publications, so there are mechanism against data drifts.
* *17 Completeness (statistically significant data to cover input space and OOD as per System/subsystem Level Requirements)*: By design the data is rather exhaustive, but missing other commercial standards that are FAA accepted (SAE, ASTM, MIL-SPEC, RTCA).
* *18 Format of dataset files*: Pandas [`dataframe`](https://pandas.pydata.org/pandas-docs/stable/reference/frame.html#dataframe).
* *19 Security tag(s)*: I don't understand.
* *20 Monitoring or Recording tag(s)*: I don't understand, but should be ensured by US gov.

# Risk-based levelling of objectives from [EASA Artificial Intelligence Concept Paper Issue 2 Guidance for Level 1 & 2 machine-learning applications](https://www.easa.europa.eu/en/downloads/139504/en)

## Trustworthiness analysis
* *ET-01:  The  applicant  should  perform  an  ethics-based trustworthiness assessment for any AI-based system developed using ML techniques or incorporating ML models.*



* *ET-02: The applicant should ensure that  the  AI-based system bears no risk of creating over-reliance, attachment, stimulating addictive behaviour, or manipulating the end user’s behaviour.*



* *ET-03: The applicant should comply with national and EU data protection regulations (e.g. GDPR), i.e. involve their Data Protection Officer (DPO), consult with their National Data Protection Authority, etc.*



* *ET-04: The applicant should ensure that the creation or reinforcement unfair bias in the AI-based system, regarding both the data sets and the trained models, is avoided, as far as such unfair bias could have a negative impact on performance and safety.*



* *ET-05: The applicant should ensure that end users are made aware of the fact that they interact with an AI- based system, and, if applicable, whether some personal data is recorded by the system .*



* *ET-06: The applicant should perform an environmental impact analysis, identifying and assessing potential negative impacts of the AI-based system on the environment and human health throughout its life cycle (development, deployment, use, end of life), and define measures to reduce or mitigate these impacts.*



* *ET-07: The applicant should identify the need for new competencies for users and end users to interact with and operate the AI-based system, and mitigate possible training gaps.*




* *ET-08: The applicant should perform an assessment of the risk of de-skilling of the users and end users and mitigate the identified risk through a training needs analysis and a consequent training activity.*



## AI Assurance
* *DA-01: The applicant should describe the proposed learning assurance process, taking into account each of the steps described in Sections C.3.1.2 to C.3.1.14, as well as the interface and compatibility with development assurance processes.*



* *DA-02: Based on (sub)system requirements that have been allocated to the AI/ML constituent, the applicant should capture the following minimum requirements for the AI/ML constituent: — safety requirements allocated to the AI/ML constituent; — information security requirements allocated to the AI/ML constituent; — functional requirements allocated to the AI/ML constituent; — operational requirements allocated to the AI/ML constituent, including AI/ML constituent ODD monitoring and performance monitoring, detection of OoD input data and data-recording requirements;
— other non-functional requirements allocated to the AI/ML constituent; and interface requirements.*



* *DA-03: The applicant should define the set of parameters pertaining to the AI/ML constituent ODD, and trace them to the corresponding parameters pertaining to the OD when applicable.*



* *DA-04: The applicant should capture the DQRs for all data required for training, testing and verification of the AI/ML constituent, including but not limited to: […]*



* *DA-05: The applicant should capture the requirements on data to be pre-processed and engineered for the inference model in development and for the operations.*



* *DA-06: The applicant should describe a preliminary AI/ML constituent architecture, to serve as reference for related safety (support) assessment and learning assurance objectives.*



* *DA-07: The applicant should validate each of the requirements captured under Objectives DA-02, DA-03, DA-04, DA-05 and the architecture captured under Objective DA-06.*



* *DA-08: The applicant should document evidence that all derived requirements have been provided to the (sub)system processes, including the safety (support) assessment.*



* *DA-09: The applicant should document evidence of the validation of the derived requirements, and of the determination of any impact on the safety (support) assessment and (sub)system requirements.*



* *DA-10: Each of the captured (sub)system requirements allocated to the AI/ML constituent should be verified.*




* *DM-01: The applicant should identify data sources and collect data in accordance with the defined ODD, while ensuring satisfaction of the defined DQRs, to drive the selection of the training, validation and test data sets.*



* *DM-02-SL: Once data sources are collected, the applicant should ensure that the annotated or labelled data in the data set satisfies the DQRs captured under Obj. DA-04.*



* *DM-02-UL: Once data sources are collected and the test data set labelled, the applicant should ensure that the annotated or labelled data in this test data set satisfies the DQRs captured under Objective DA-04.*



* *DM-03: The applicant should define the data preparation operations to properly address the captured requirements (including DQRs).*



* *DM-04: The applicant should define and document pre- processing operations on the collected data in preparation of the model training.*



* *DM-05: When applicable, the applicant should define and document the transformations to the pre-processed data from the specified input space into features which are effective for the performance of the selected learning algorithm.*



* *DM-06: The applicant should distribute the data into three separate data sets which meet the specified DQRs in terms of independence (as per Objective DA-04): —  the training data set and validation data set, used during the model training; —   the test data set used during the learning process verification, and the inference model verification.*



* *DM-07: The applicant should ensure validation and verification of the data, as appropriate, throughout the data management process so that the data management requirements (including the DQRs) are addressed.*



* *DM-08: The applicant should perform a data verification step to confirm the appropriateness of the defined ODD and of the data sets used for the training, validation and verification of the ML model.*



* *LM-01:  The  applicant  should  describe  the  ML  model architecture.*



* *LM-02: The applicant should capture the requirements pertaining to the learning management and training processes, including but not limited to: —   model family and model selection; —   learning algorithm(s) selection; —   explainability capabilities of the selected model; —   activation functions; —   cost/loss function selection describing the link to the performance metrics; —   model  bias  and  variance  metrics  and  acceptable levels; —   model    robustness    and    stability    metrics    and acceptable levels; —   training   environment   (hardware   and   software) identification; —   model parameters initialisation strategy; —   hyper-parameters identification and setting; —   expected performance with training, validation and test sets.*



* *LM-03: The applicant should document the credit sought from the training environment and qualify the environment accordingly.*



* *LM-04:   The   applicant   should   provide   quantifiable generalisation bounds.*



* *LM-05: The applicant should document the result of the model training.*



* *LM-06: The applicant should document any model optimisation that may affect the model behaviour (e.g. pruning, quantisation) and assess their impact on the model behaviour or performance.*



* *LM-07-SL: The applicant should account for the bias- variance trade-off in the model family selection and should provide evidence of the reproducibility of the model training process.*



* *LM-08: The applicant should ensure that the estimated bias and variance of the selected model meet the associated learning process management requirements.*



* *LM-09: The applicant should perform an evaluation of the performance of the trained model based on the test data set and document the result of the model verification.*



* *LM-10:  The  applicant  should  perform  requirements- based verification of the trained model behaviour.*



* *LM-11: The applicant should provide an analysis on the stability of the learning algorithms.*



* *LM-12: The applicant should perform and document the verification of the stability of the trained model, covering the whole AI/ML constituent ODD.*



* *LM-13: The applicant should perform and document the verification of the robustness of the trained model in adverse conditions.*



* *LM-14:  The  applicant  should  verify  the  anticipated generalisation bounds using the test data set.*



* *LM-15: the applicant should capture the description of the resulting ML model.*



* *LM-16: The applicant should confirm  that the trained model verification activities are complete.*



* *IMP-01: The applicant should capture the requirements pertaining to the ML model implementation process.*



* *IMP-02: The applicant should validate the model description captured under Objective LM-15 as well as each of the requirements captured under Objective IMP- 01.*



* *IMP-03: The applicant should document evidence that all derived requirements generated through the model implementation process have been provided to the (sub)system processes, including the safety (support) assessment.*



* *IMP-04: Any post-training model transformation (conversion, optimisation) should be identified and validated for its impact on the model behaviour and performance, and the environment (i.e. software tools and hardware) necessary to perform model transformation should be identified.*



* *IMP-05: The applicant should plan and execute appropriate development assurance processes to develop the  inference model into software and/or hardware items.*



* *IMP-06: The applicant should verify that any transformation (conversion, optimisation, inference model development) performed during the  trained model implementation step has not adversely altered the defined model properties.*



* *IMP-07: The differences between the software and hardware of the platform used for training and those used for verification should be identified and assessed for their possible impact on the inference model behaviour and performance.*



* *IMP-08: The applicant should perform an evaluation of the performance of the inference model based on the test data set and document the result of the model verification.*



* *IMP-09: The applicant should perform and document the verification of the stability of the inference model.*



* *IMP-10: The applicant should perform and document the verification of the robustness of the inference model in adverse conditions.*



* *IMP-11: The applicant should perform requirements- based verification of the inference model behaviour when integrated into the AI/ML constituent.*



* *IMP-12: The applicant should  confirm  that  the AI/ML constituent verification activities are complete.



* *CM-01: The applicant should apply all configuration management principles to the AI/ML constituent life- cycle data, including but not limited to: —   identification  of  configuration   items;  versioning; baselining; —   change control; reproducibility; —   problem reporting; —   archiving and retrieval, and retention period.*



* *QA-01: The applicant should ensure that quality/process assurance principles are applied to the development of the AI-based system, with the required independence level.*



* *RU-01: The applicant should perform an impact assessment of the reuse of a trained ML model before incorporating the model into an AI/ML constituent. The impact assessment should consider: […]*



* *RU-02: The applicant should perform a functional analysis of the COTS ML model to confirm its adequacy to the requirements and architecture of the AI/ML constituent.*



* *RU-03: The applicant should perform an analysis of the unused functions of the COTS ML model, and prepare the deactivation of these unused functions.*



* *SU-01: The applicant should capture the accuracy and fidelity of the reference model, in order to support the verification of the accuracy of the surrogate model.*



* *SU-02: The applicant should identify, document and mitigate the additional sources of uncertainties linked with the use of a surrogate model.*



* *EXP-01: The applicant should identify the list of stakeholders, other than end users, that need explainability of the AI-based system at any stage of its life cycle, together with their roles, their responsibilities and their expected expertise (including assumptions made on the level of training, qualification and skills).*



* *EXP-02: For each of these stakeholders (or groups of stakeholders), the applicant should characterise  the need for explainability to be provided, which is necessary to support the development and learning assurance processes.*



* *EXP-03: The applicant should identify and document the methods at AI/ML item and/or output level satisfying the specified AI explainability needs.*



* *EXP-04: The applicant should design the AI-based system with the ability to deliver an indication of the level of confidence in the AI/ML constituent output, based on actual measurements or on quantification of the level of uncertainty.*



* *EXP-05: The applicant should design the AI-based system with the ability to monitor that its inputs are within the specified operational boundaries (both in terms of input parameter range and distribution) in which the AI/ML constituent performance is guaranteed.*



* *EXP-06: The applicant should design the AI-based system with the ability to monitor that its outputs are within the specified operational performance boundaries.*



* *EXP-07: The applicant should design the AI-based system with the ability to monitor that the AI/ML constituent outputs (per Objective EXP-04) are within the specified operational level of confidence.*



* *EXP-08: The applicant should ensure that the output of the specified monitoring per the previous three objectives are in the list of data to be recorded per MOC EXP-09-2.*



* *EXP-09: The applicant should provide the means to record operational data that is necessary to explain, post operations, the behaviour of the AI-based system and its interactions with the end user, as well as the means to retrieve this data.*




## Human factors for AI
* *EXP-10: For each output of the AI-based system relevant to task(s) (per Objective CO-02), the applicant should characterise the need for explainability.*



* *EXP-11: The applicant should ensure that the AI-based system presents explanations to the end user in a clear and unambiguous form.*



* *EXP-12: The applicant should define relevant explainability so that the receiver of the information can use the explanation to assess the appropriateness of the decision / action as expected.*



* *EXP-13: The applicant should define the level of abstraction of the explanations, taking into account the characteristics of the task, the situation, the level of expertise of the end user and the general trust given to the system.*



* *EXP-14: Where a customisation capability is available, the end user should be able to customise the level of abstraction as part of the operational explainability.*



* *EXP-15: The applicant should define the timing when the explainability will be available to the end user taking into account the time criticality of the situation, the needs of the end user, and the operational impact.*



* *EXP-16: The applicant should design the AI-based system so as to enable the end user to get upon request explanation or additional details on the explanation when needed.*



* *EXP-17: For each output relevant to the task(s), the applicant should ensure the validity of the specified explanation.*



* *EXP-18: The training and instructions available for the end user should include procedures for handling possible outputs of the ODD monitoring and output confidence monitoring.*



* *EXP-19: Information concerning unsafe AI-based system operating conditions should be provided to the end user to enable them to take appropriate corrective action in a timely manner.*



* *HF-01: The applicant should design the AI-based system with the ability to build its own individual situation representation.*



* *HF-02: The applicant should design the AI-based system with the ability to reinforce the end-user individual situation awareness.*



* *HF-03: The applicant should design the AI-based system with the ability to enable and support a shared situation awareness.*



* *HF-04: If a decision is taken by the AI-based system that requires validation based on procedures, the applicant should design the AI-based system with the ability to request a cross-check validation from the end user.*



* *HF-05: For complex situations under normal operations, the applicant should design the AI-based system with the ability to identify a suboptimal strategy and propose an improved solution. Corollary objective: The applicant should design the AI- based system with the ability to process and act upon a proposal rejection from the end user.*



* *HF-06: For complex situations under abnormal operations, the applicant should design the AI-based system with the ability to identify the problem, share the diagnosis including the root cause, the resolution strategy and the anticipated operational consequences. Corollary objective: The applicant should design the AI- based system with the ability to process and act upon arguments shared by the end user.*



* *HF-07: The applicant should design the AI-based system with the ability to detect poor decision-making by the end user in a time-critical situation, alert and assist the end user.*



* *HF-08: The applicant should design the AI-based system with the ability to propose alternative solutions and support its positions.*



* *HF-09: The applicant should design the AI-based system with the ability to modify and/or accept the modification of task allocation pattern (instantaneous/short-term).*



* *HF-10: If spoken natural language is used, the applicant should design the AI-based system with the ability to process end-user requests, responses and reactions, and provide indication of acknowledged user’s intentions.*



* *HF-11: If spoken natural language is used, the applicant should design the AI-based system with the ability to notify the end user that he or she possibly misunderstood the information.*



* *HF-12: If spoken natural language is used, the applicant should design the AI-based system with the ability to identify through the end-user responses or his or her action that there was a possible misinterpretation from the end user.*



* *HF-13: In case of confirmed misunderstanding or misinterpretation of spoken natural language, the applicant should design the AI-based system with the ability to resolve the issue.*



* *HF-14: If spoken natural language is used, the applicant should design the AI-based system with the ability to not interfere with other communications or activities at the end user’s side.*



* *HF-15: If spoken natural language is used, the applicant should design the AI-based system with the ability to provide information regarding the associated AI-based system capabilities and limitations.*



* *HF-16: If spoken procedural language is used, the applicant should design the syntax of the spoken procedural language so that it can be learned and applied easily by the end user.*



* *HF-17: If gesture language is used, the applicant should design the gesture language syntax so that it is intuitively associated with the command that it is supposed to trigger.*



* *HF-18: If gesture language is used, the applicant should design the AI-based system with the ability to disregard non-intentional gestures.*



* *HF-19: If gesture language is used, the applicant should design the AI-based system with the ability to recognise the end-user intention.*



* *HF-20: If gesture language is used, the applicant should design the AI-based system with the ability to acknowledge the end-user intention with appropriate feedback.*



* *HF-21: In case spoken natural language is used, the applicant should design the AI-based system so that this modality can be deactivated for the benefit of other modalities.*



* *HF-22: If spoken (natural or procedural) language is used, the applicant should design the AI-based system with the ability to assess the performance of the dialogue.*



* *HF-23: If spoken (natural or procedural) language is used, the applicant should design the AI-based system with the ability to transition between spoken natural language and spoken procedural language, depending on the performance of the dialogue, the context of the situation and the characteristics of the task.*



* *HF-24: The applicant should design the AI-based system with the ability to combine or adapt the interaction modalities depending on the characteristics of the task, the operational event and/or the operational environment.*



* *HF-25: The applicant should design the AI-based system with the ability to automatically adapt the modality of interactions to the end-user states, the  situation, the context and/or the perceived end user’s preferences.*



* *HF-26: The applicant should design the AI-based system to  minimise  the  likelihood  of  design-related  end-user errors.*



* *HF-27: The applicant should design the AI-based system to minimise the likelihood of HAIRM-related errors.*



* *HF-28: The applicant should design the AI-based system to be tolerant to end-user errors.*



* *HF-29: The applicant should design the AI-based system so that in case the end user make an error while interacting with AI-based system, the opportunities exist to detect the error.*



* *HF-30: The applicant should design the AI-based system so that once an error is detected, the AI-based system should provide efficient means to inform the end user.*





## Safety risk mitigation
* *SRM-01: Once activities associated with all other building blocks are defined, the applicant should determine whether the coverage of the objectives associated with the explainability and learning assurance building blocks is sufficient  or  if an additional dedicated layer  of protection, called hereafter safety risk mitigation, would be necessary to mitigate the residual risks to an acceptable level.*



* *SRM-02: The applicant should establish safety risk mitigation means as identified in Objective SRM-01.*



## Additional risk-based levelling of information-security-related objectives
* *IS-01: For each AI-based system and its data sets, the applicant should identify those information security risks with an impact on safety, identifying and addressing specific threats introduced by AI/ML usage.*



* *IS-02: The applicant should document a mitigation approach to address the identified AI/ML-specific security risk.*



* *IS-03: The applicant should validate and verify the effectiveness of the security controls introduced to mitigate the identified AI/ML-specific security risks to an acceptable level.*



<!--
# TODO
* Reorganiser le cod le code
* Relancer tout
* faire la database de texte
* faire des use case?
  * classification? classification en utilisant DRS Document Types Metadata Mapping(1) => stc_collection_test_offset=1250.xlsx
  * generation?
* pdf to text avec google vision pour tous les docs
* qui limite à une certaine mod d'helico
* trouver un vrai use case avec lequel il y a un vrai pattern
* statistics on desctiption legnth (# of words)
* statistics on Limitations and Condition (# of words and sentences)
* classifier pour les ATA, avec les ATA les plus classiques
* STC content (limitations/correl;ations):
  * verifier si les deux sont anticorr'lé
  * améliore l'algo d'extraction des limitatiosn te confdisiotn
* faire rounite d'extraction de paragraphes -> 92
  * outline du pdf
  * regex https://regex101.com/
  * figures
  * si bookmark correspiond pas a un vrai titre
  * faire nested liste ou ien juste flat avec toute l'arbo...
  * faire un regex pour extraire le mocreau de texte.
  * pattern utiles pour detection de title (initial):
    * extraction de la ToC depuis pdf plnuimber
    * Excellent:
      * début de phrase
      * Phrase avec peu de mots
      * parttern tres recurent suitnt le type de doc
    * Bon marqueur:
      * En gras
      * majuscule
      * deux points
    * Moyen marqueur:
      * keyword connus
      * point final 
    * En vrac:
      * En gras
      * keyword connus
      * peu de mots
      * fini sans un point final
      * majuscule
      * saut de ligne avant, mais pas apres
      * parttern tres recurent suitnt le type de doc
      * des fois des deux points
    * Examples:
      * (6) Training on Records.
      * 21. PASSENGER AND CARGO LOADING PROCEDURES. 
      * (1) Loading, based on aircraft configuration, i.e.,....
      * 1. Purpose.
      * Purpose: This InFO informs air carriers of the availability of the Department of Transportation’s (DOT) National Aviation Resource Manual for Quarantinable Diseases. 
      * SUBJECT: DOT

## TODO:
* STC content (limitations/correl;ations):
  * refaire une analyse avec google vision sur fichiers qui ont d'econne seulement: le faire avec les filters. Se faire un fichier excel pour releceture plus simple
* Identify pdfs requiring google vision by type: STC, CAR, 
* Retrieve additional attachment (xml, html, doc, docs, xls, xlsx). Not sure since not additional info.
* faire classifier d'ATA et l'appliquer sur les STC
* identifier un use case pour le STC:
  * pas simple, pas vraiment d'apprentissage car peu de lien entre les différents paragraphes
  * OK pour statistique explicatives:
    * où est-ce qu'il y a les ICA et FMS
    * où est-ce que la hrase générique d'incompatibilité
    * lien entre ACO et ODA
    * conbien listent les ICA et FMS en lim vs conteniu du design change
    * diversité des l;imitationms listées
    * limitation à une mod d h;élico
  * faire de la génération? 
* Use case sur tout le DRS:
  * faire classifier général sur tout avec remplacement de mots
  * découper les paragraphes
  * un bon use case est de bosser sur des bons doc qui sont des guidance
* Use pour un sous-ensemble du DRS:
  * doc de qualit'e: order, AC, 
  * outline/bookmark dispo
  * doc a partir desquels on peut faire decoupage question/reponse par paragrahe
* Check compliance against 1000.36 -> needs to be translated from rules to examples.
 * peut-etre que les modeles existants permettent de faire du zero-shot dessus?
 -->