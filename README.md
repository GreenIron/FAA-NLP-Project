# A NLP project on FAA documents

## Abstract
This is an in-work personal NLP project dealing with FAA documentation: use case definition, data source identification, data collection, data cleaning, data mining and some RAG and LLM fine-tuning.

# Introduction

## Motivation
While aviation is a heavy producer of documentation, most of the aviation-related database publicly available on popular ML websites (kaggle, huggingface, google data) is about unformatted tabular data (e.g. accident rates, airport traffic, helicopter types) and not about NLP. In the same time legal has had a several datasets available.
This is quite unusual consedering that aviation data is very good at producing complex, high-quality and standardized documents or more generally any sorts of communication (e.g. GAMA standards, ATC standards).

While it takes at least 10 years for a commercial-level technology to hit aviation, data has been at the core of airborne aviation for a lot quite some times (CVFDR, telemetry...).
FAA has been offering for several years a moderm and powerful platform to browse all the publicly available FAA documentation: [DRS](https://drs.faa.gov/). This platform offers tremendous capabilities and an API (one of the many DOT platforms) to programaticaly acccess its documentation.

I truely encourage you to take a look at it right now since this will be the primary source of data.

Non governmetal platforms like [AHST](https://ushst.org/) offer also high quality documentation, but have no API.

## Why aviation data
The starting block of any ML project is to get to intimately familiar with the data. Since I work in aviation (rotorcraft), I've a professional interest in knowing more about all types of documents (format and conten)t that the FAA produces (with a focus on rotorcrafts).

From a pure ML perspective, aviation data offers lots of benefits:
* because of the safety constraints, aviation structuraly produces lots of data in various format 
* aviation data is diversifed (e.g. produced by people with various background) and standardized (FAA Forms\, Tech logs, ATC communication, ARINC 429 GAMA, accident reports...)
* aviation data has been around for many decades

And more specifically for the text documents released by the FAA:
* FAA offers a powerful API
* FAA documents are well formatted (Q&A tables, consistent tables of content)
* FAA documents have a high quality standard (Order 1000.36) compared to the best NLP dataset that one can find on Hugging Face

# Data Collection
## Results on data collection
DOT APIs are very easy to use and these are the steps that I followed to collect DRS data:
1. Get familiar with the API: https://drs.faa.gov/help/helpdetails
2. Download the DRS Document Types Metadata Mapping file from https://drs.faa.gov/help/helpdetails
3. Collect the index table for each DRS doc type (~ 1 hour)
4. Collect each file or text from each document found in each index table (several months)

This is pretty straghtforward, but I hit several hard points:
1. DRS website changed a little bit, so the code had to follow
2. Some files caused some issue (filename with special characters like &)
3. After some times my IP address got banned, so I started using a VPN to finish my data collection
4. On top of the main file, DRS offers also "Attachments/Public Comments" documents. As of today those are not searchable with DRS and not in the DRS Document Types Metadata Mapping file.
5. There also non-pdf and non-text files

Note that Aeronautical Information Manual (AIM) and Aeronautical Information Publication (AIP) are two basic references in aviation. They are not in DRS, so I've added them manually to the dataset.

## Other sources of data worth exploring
The following data could also be an excellent source of data for an NLP project. I've not had the time to focus too much on those, but they would be well suited for a simple use case:
* public comments are organized as Q&A tables. They are generally very nicelly formatted so that a tabular extrator libraty should work (sometimes alswer formatted in csv like files). The answer provided by the FAA is not "right" like what can be found in other factual Q&A database, but this could still be a good starting point.
* AC 43-16A - General Aviation Maintenance Alerts offers also an excelent source of maintenance database
* MMELs are also well formatted, but definitely less diversified
* FAA DER training center (learning documents and Q&A)
* all the DRS missing data and all the documents which were discontinued to exist

# A first try with STC (Supplemental Type Certificate)

## Why STC certificates
STC is the master document that defines the type design of an aircract modification, it's the final document (except for the PMA) that gets issued at the end of a certification project. I happen to be working a lot with STCs, so I knew that those enjoyed lots of nice properties:
* small (<1000 tokens so local LLM can be fine-tuned)
* rigid structure
* informative
* diverse
* long history 

## Metadata only (with a focus on rotorcrafts)
The DRS metadadata already offers a wealth of information before even starting exploring the text content. I've only analysed a fraction of the metadata and the following is another fraction of this analysis (there is more in the different Python use cases). It's really a high-level analysis to get to know the data better, but I didn't push towards more complex data mining methods (dimensiality reduction, clustering... scikit-learn is the best source I know to start a ML project). Maybe something for later.

The metadata file has as many rows for an STC as there are issuances. Date of STC (re)-issuance is given by "stcStatusDate". A STC is reissued for example to add a new configuration (e.g. obsolescence). Note that a STC can be updated (e.g. minor change for editorial changed) and not reissued and in that case the latest update will be indicated in the field "docLastModifiedDate".
"drs:stcAplicationDate" corresponds to the application date for the firs STC issuance.

* Number of STC (documents):
  * indexed: 76416
  * with PDF (usable): 43818

* Sanity check:
  * only one broken PDF ([SA00765DE](https://drs.faa.gov/browse/excelExternalWindow/2F49BD200CEC24DD86257F5600610445.0001?modalOpened=true))
  * very few abnormal dates (<50)
  * very few headers typos for the DRS metadadata ("Document Type name in DRS ", "Deafult  Sort By")

* *Rotorcraft STCs reissuance distribution*: there is a period (which is remarkable) that is ~3 years, but looking more closely revelas that the actual period is more like 1.6 years (there is another peak around 7 yeahs (~1.7*4)). It could be interesting to filter by ATA, company, ACO... to try to consolidate or explain this pattern. Another other interesting observation is that is period is not consitent when we switch to airplanes.

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

* *Helicopter STCs by holder between 1990 and 2000*: a big surprise to me was that American Eurocopter (now Airbus Helicopters, Inc.) was one of the largest issuers of STC certificates (but a detailed analysis of the STCs show that those are not complex) while still reamaining a rather small company. It's also worth noting that the different companies have very different business models (ODA doing worth for third parties, ODA doing only internal work, companies with core products working with ACO).

![plot](./images/20_STC_by_STC_holder_helicopter_1900.png)

* *Historical and Current STCs*: the distribution is quite expected, but most surprising is that the distribution is not reflecting the relative weight of those markets (large multi-engine should be significantly highier).

![plot](./images/20_STCs_by_stcProductSubType.png)

I could keep going on and on (e.g. looking at time drifts, comparing single engine vs. multi-engines, focusing on a region or on a company, crossing with other databasis...), but that's enough histograms for today. More details are in the jupiter notebooks.

## Text analysis
### Getting to know the data
STC textual data was decoded using a mix of from pypdf and PyPDF2 for pure PDF decoding and Google Cloud Platform OCR when only a scan was available.
GCP OCR proved to quite uneffective on a fraction of scanned pdfs (SA00005MC-D) yielding very odd artifacts (unable to decode some letters, paragraphs in the wrong order). GCP OCR does not offer any options to adjust performances, so the only workaround was to perform some image preprocessing (was able to improve performance).

STCs are well defined and the format has remained pretty much the same over the years. The key fields from a text standpoint are the "Descriptions" and the "Limitations/Conditions". The rest of the data is quite redundant with the metadata. Let's look at some examples first:
* accross the ages: [old](https://drs.faa.gov/browse/excelExternalWindow/BA172489A7E9DA1586257CD30047CF76.0001?modalOpened=true) , [recent](https://drs.faa.gov/browse/excelExternalWindow/DRSDOCID128075530320230314172310.0001)
* description: [short](https://drs.faa.gov/browse/excelExternalWindow/CA21490D878D3C63862574C600496658.0001?modalOpened=true), [average](https://drs.faa.gov/browse/excelExternalWindow/3FE1351336C0748F8625831400585C06.0001?modalOpened=true), [long](https://drs.faa.gov/browse/excelExternalWindow/17914C2EEDFFC8C98625804300770386.0001?modalOpened=true)
* limitations/Conditions: [short](https://drs.faa.gov/browse/excelExternalWindow/DRSDOCID125280334620221128154006.0001?modalOpened=true), [average](https://drs.faa.gov/browse/excelExternalWindow/3FE1351336C0748F8625831400585C06.0001?modalOpened=true), [long](https://drs.faa.gov/browse/excelExternalWindow/EF198F621A3B2AA9862585D90044ACBF.0001?modalOpened=true)

The following two pictures show the word distributions for those two fields. Description is a Gama distribution, while Limitations/Conditions is more multimodal. That's not surprising since the Description is more formatted, while the Limitations/Conditions fields is often used a field for secondary informations.

![plot](./images/22_number_of_words_in_description.png)

![plot](./images/22_number_of_words_in_limitations.png)

### (Lack of) Results
I failed to find a decent LLM application:
* zero shot using 7B LLMs (Ollama) yielded poor results (guessing meta data based on Description, Limitations/Conditions).
* ATA-classification (using DRS MMEL and JASC) performances were poor.
* Overall I could not find any interresting use cases (e.g. guess the Description based on Limitations/Conditions).

The key problems might be that the STC Description and Limitations/Conditions are not informative enough and are quite independent between each others from the metadata (there are no patterns to learn or represent). Another problem is that the Description is often based on the commercial name of the modification/equipment, which is too much of a specific usage domain. Unlike a catalogue, it is not meant to provide a commercial description. We would need to scrap data from all STC holders, correlate the P/N with the catalogue description to have more information.

Following picture shows the lack of correlation between Description and a Limitations/Conditions. It's worth noting that there are some obvious vertical and horizontal bars. Those might be linked to "standard sentences" ([AC No: 20-188](https://www.faa.gov/documentlibrary/media/advisory_circular/ac_20-188.pdf)). 

![plot](./images/22_limitations_vs_descriptions.png)

Other than that my main findings are that:
* There is some overlap between what's a Description vs. a Limitations/Conditions. For example the RFMS and ICA can be found in one on the other.
* Limitations/Conditions is sometimes used for configuration control.
* There is a wide variability on how those fields are used (depth of explainations).

# All DRS and the other non DRS documents

## Data collection
It took me about 3 months to collect all DRS data on my personal computer (working 1 or 2 hours before going to bed, so not a full time job), but many more months to set up a functional data collection routine. I collected all the DRS data, plus a few more (AIM, AIP). There was no signigicant difficulties, except being patient and improving the code to come up with any exceptions found on DRS (e.g. new data format, attachments, time-out, pdf crashes). 
The whole raw documents dataset (mainly pdf, but also doc, xls, html) is about 55GB.

## Getting to know the data
At the time of my data collection, there was:
* ~172799 indexed documents
* ~324805 available documents if we add the attachments. Those include a lot of redundant documents, because DRS makes available doc and html versions of some pdfs.
* ~171254 documents that can be decoded without OCR (I didn't try to OCR all the missing documents for budget reasons). Non-decodable OCR also include documents with no content (only the index is available).

This dataset is currently qui airplanes/rotorcrafts-oriented, but the FAA is adding more and more data for space travels.

The following pdf made PdfReader crash really bad, but I didn't investigate further:
* [CAM 1: Supplement No. 1; New Issuance System - Change of date of CAM 1 to December 15, 1959](https://drs.faa.gov/browse/excelExternalWindow/FAA000000000000000CAM1_121559PDF.0001?modalOpened=true)
* [CAM 4b: Supplement No. 1; Certification and Operation of Certain Airplanes forthe Department of the Interior in the Trust Territory of the Pacific Islands](https://drs.faa.gov/browse/excelExternalWindow/FAA00000000000000CAM4b_050160PDF.0001?modalOpened=true)
* [CAM 51: Supplement No. 2; 51.17 - Standard of Performance](https://drs.faa.gov/browse/excelExternalWindow/FAA00000000000000CAM51_060851PDF.0001?modalOpened=true)

I skipped the metadata analysis and focused only a the tokens to have an idea of what NLP tool to use.

The following table recaps the number of documents by DRS document type:
DRS doc types|ADFRAWD|ADNPRM|AC|AB|POLICY|CAM|CAR|CANIC|ADFREAD|ELOS|EXEMPTION|CFRFRSFAR|NORSEE|NPRM|PMA|SAIB|SCFINAL|SCPROPOSED|SFAR|STC|TSOI|TSO|FAR|TCDSMODEL|UNAPPROVED_PARTS_NOTIFICATIONS|ENVIRONMENTAL_SUPPORTING_DOCUMENTS|LAUNCH_SITE_OPERATOR_LICENSES|LAUNCH_VEHICLE_OPERATOR_LICENSES|PERMITS|REENTRY_SITE_OPERATOR_LICENSES|REENTRY_VEHICLE_OPERATOR_LICENSES|SAFETY_ELEMENT_APPROVALS|ORDER_8900.1|CLARIFY_POLICY|AFS-1_MEMORANDUMS|AIRCRAFT_MASTER_SCHEDULE|AIRCRAFT_STANDARDIZED_CURRICULUM|AT_JTA|OTHER_AWO|ALERTS|OTHER_AWARDS_INFORMATION_GUIDES|8900.1_EDITORIAL_CORRECTIONS|OTHER_EFB_RESEARCH_REPORTS|OTHER_EFB_CHECKLISTS|OTHER_FAA_90_DAY_SAFETY_REVIEW|OTHER_CPD_6.03|AFS_FFS_UPDATEPUB|AFS_FFS_UPDATES|FOEB|FSB_REPORTS|AFS_POLICY_DEV_MEMOS|OTHER_FLIGHT_STANDARDS_ORM_WORKSHEETS|AFS_FOCUS_TEAMS|GA_JTA|BULLETINS|OTHER_PS_HANDBOOKS|INFO|OTHER_INFORMATION_GUIDES|PILOT_QUALIFICATION_CURRICULUM|OTHER_INTERNATIONAL_PUBLICATIONS|OTHER_JOB_AIDS|OTHER_LASER_INVESTIGATION_REFERENCES|MMEL_POLICY_LETTERS|MMEL|NOTICES|OSR|OPSS_GUIDANCE|OSWG|ORDER_8300.10|ORDER_8400.10|ORDER_8700.1|ORDER_8740.1|ORDERS|PART_129_OPSPEC_JA|OTHER_PS_FEDERAL_AVIATION_ACTS|OTHER_PS_LEGAL_INTERPRETATIONS|OTHER_PS_POLICY_MEMORANDA|OTHER_PS_PREAMBLES|OTHER_QMS_AND_BP|OTHER_RCCB|OTHER_RIRTP|SAFO|SAS_AXH_DCT|SAS_DCT|STCRELIEFAPPLETTER|8900.1_SUMMARY_OF_CHANGES|OTHER_SPRS|OTHER_VPM|LEGAL_INTERPRETATIONS
---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---
Number of documents|46666|15168|4212|20|1489|992|4966|0|730|3888|57658|4745|30|1035|56999|2682|2893|736|176|0|7137|1186|24255|15389|383|72|30|76|22|0|0|14|16832|0|0|3|3|2244|0|429|56|238|12|4|2|0|23|0|501|1377|0|0|29|1801|8643|588|499|29|3|40|17|0|872|10397|5133|94|58|490|1045|1311|653|181|4478|48|172|0|0|0|0|18|0|388|1073|5172|683|3441|0|0|2076

I wanted to analyse the documentary inflation over the years, but I stopped since it would be wrongly interpreted. 

* *Token distribution for all DRS*: As expected the distribuytion is multimodal with a heavy tail. It's interresting to note that the average size is rather small and could be entirely fed to a local LLM (~1k). This limitation does not apply to GAFAM provided LLM (100k to 1M tokens).

![plot](./images/27_token_distr_all_DRS.png)

* *Token distribution for ADFINAL*: this one present a very odd distribution with its two bell-shaped multimodal. It woulbe be interresting to understand if those two modes correspond to two different patterns.

![plot](./images/27_token_distr_adfinal.png)

* *Token distribution for AC:* I was more surprised to see that the AC follows a classic exponential decrease, because I'm used to very very long ACs (AC27 and AC20).

![plot](./images/27_token_distr_ac.png)

* *Token distribution for MMEL:* I was expecting a distribution concentrated around a single mode, but the MMEL distribution is more complex.

![plot](./images/27_token_distr_mmel.png)

* *Token distribution for ORDERS:* Same comment as AC.

![plot](./images/27_token_distr_orders.png)

## ML applications
The size of the dataset makes the project intractable on my personal computer, so I decided to narrow-down the usage domain.

As far as I know there are several ways to work on a document with LLM: pass the document in the context window (extremely resource intensive), fine-tune (very resource intensive), RAG (manageable).

role: adapt role de' exper / task:  review against standard and rpviode advice / context: target audience, background information, scenario

### AC-ORDER-HANDBOOK-AIP-AIM
Blabla. Segmentation.

### RAG-use case

#### Why RAG
Short intro based on https://haystack.deepset.ai/, [nlp book](https://www.oreilly.com/library/view/natural-language-processing/9781098136789/)

[langchain](https://www.langchain.com/), [LlamaIndex](https://www.llamaindex.ai/), [Pinecone](https://www.pinecone.io/), [Weaciate](https://weaviate.io/)

## RAG

### LLM fine tuning

[check this](https://generallyintelligent.substack.com/p/fine-tuning-mistral-7b-on-magic-the)



<!-- 
https://helixml.substack.com/p/how-we-got-fine-tuning-mistral-7b
-->

### Zero-shot with Gemini 1.5
Gemini 1.5 provides a 1.5M todek window, paving the way...

# ARP6983 Criteria Review

### MLDL objectives

#### 1 ML Constituent ODD
* *1.1 The MLCODD is characterized*: Yes. The ODD is actually explicitly defined in the dataset itself for most of it. Otherwise it is detailed on the FAA website.
* *1.2 AN ML Data impact analysis is developed from the MLCODD*: Unclear?
* *1.3 Any limitation of use to the ML-based System/Subsystem processes is provided, including the System Safety Assessment process*: None.

#### 2 ML Constituent Requirements
* *2.1 ML Constituent requirements are developed for each ML Constituent*: 
* *2.2 Derived ML Constituent requirements are defined and provided to the ML-based System/Subsystem processes, including the System Safety Assessment process*:

#### 3 ML Environment Set Up
* *3.1 The MLDL environment is selected and defined*: Based on conda [`environment.yml`](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#activating-an-environment).

#### 4 ML Data Management
* *4.1 Data sources are identified and selected*: Yes (DRS).
* *4.1 Lack of undesirable bias is ensured*: Should be N/A.
* *4.3 Data required for training, validation, and test datasets is collected*: Yes (DRS).
* *4.4 Data has specified characteristics (such as representativeness, sufficiency, fairness, etc.)*: Yes (DRS).
* *4.5 Data are cleaned up and enhanced and can be used by the ML algorithm without any additional checking*: Yes (basic cleaning). One difficulty is that the raw data (pdf) are not usable as is and need to be decoded.
* *4.6 Requirements allocated to the ML Data are satisfied*:
* *4.7 Data pre-processing description is updated*:
* *4.8 Data leakage is avoided*: Yes (ensured by data source)*:
* *4.9 Input dataset is divided into three datasets for training, validation, and test*:

#### 5 ML Model Design Model
* *5.1 The ML Model architecture is developed from the ML Constituent requirements*:
* *5.2 ML Model requirements are developed from the ML Constituent requirements and the ML  environment*:
* *5.3 Derived ML Model requirements and their rationale are defined and provided to the ML Constituent requirements process*:
* *5.4 The ML Model is built, trained, and optimized from the ML Constituent requirements and if applicable from the ML Model requirements*:
* *5.5 ML Model Description is developed from the ML Model to describe its architecture, its hyperparameters and parameters, its analytical/algorithmic syntax and semantic, its replication criteria, and its execution environment*:

#### 6 ML Validation
* *6.1 ML Model requirements comply with system/subsystem requirements*:
* *6.2 ML Model requirements are accurate and consistent*:
* *6.3 ML Model requirements are compatible with the target computer*:
* *6.4 Data processing description is compatible with the target computer*:
* *6.5 ML Model requirements are verifiable*:
* *6.6 ML Model requirements conform to standards*:
* *6.7 ML Model requirements are traceable to system/subsystem requirements*:
* *6.8 ML Model Architecture Verification*:
* *6.9 The ML architecture is compatible with the ML Model requirements*:
* *6.10 The ML architecture is consistent*: I don't understand.
* *6.11 The ML architecture is verifiable*:
* *6.12 The ML architecture conforms to standards*: Yes, RAG architecture.

#### 7 ML Model Verification
* *7.1 The ML Model complies with the ML Model requirements*: [impossibble since Hallucination is Inevitable: An Innate Limitation of Large Language Models](https://arxiv.org/abs/2401.11817)
* *7.2 The ML Model is accurate and consistent*:
* *7.3 The ML Model is compatible with target computer*: No target computer.
* *7.4 The ML Model is verifiable*:
* *7.5 The ML Model conforms to standards*: Yes (reuse of state-of-the-art RAG).
* *7.6 Verification of verification procedures*:
* *7.7 Test procedures are correct*:
* *7.8 Test results are correct, and discrepancies explained*:
* *7.9 Test coverage of ML Data and Model requirements is achieved*:
* *7.10 Test coverage of ML Model structure to the appropriate coverage criteria is achieved*:

### Data quality learning assurance objectives
* *1 Nature of data (explicit definition of input variables)*: Textual with metadata (dates and categorical)*:
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

# Conclusion



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