# Design Assurance

## Introduction

### Minimum information to present to the regulator
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

## [ARP6983](https://www.sae.org/standards/content/arp6983/) Criteria Review

### MLDL objectives

### 1 ML Constituent ODD
* *1.1 The MLCODD is characterized*: Yes. The ODD is actually explicitly defined in the dataset itself for most of it. Otherwise it is detailed on the FAA website.

### 3 ML Environment Set Up
* *3.1 The MLDL environment is selected and defined*: Based on conda [`environment.yml`](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#activating-an-environment).

### 4 ML Data Management
* *4.1 Data sources are identified and selected*: Yes (DRS).
* *4.5 Data are cleaned up and enhanced and can be used by the ML algorithm without any additional checking*: Yes (basic cleaning). One difficulty is that the raw data (pdf) are not usable as is and need to be decoded.

### 7 ML Model Verification
* *7.1 The ML Model complies with the ML Model requirements*: [impossibble since Hallucination is Inevitable: An Innate Limitation of Large Language Models](https://arxiv.org/abs/2401.11817)
* *7.4 The ML Model is verifiable*: It's minimized, see [Retrieval Augmentation Reduces Hallucination in Conversation](https://arxiv.org/abs/2104.07567) and 
[Chain-of-Thought Prompting Elicits Reasoning in Large Language Models](https://arxiv.org/abs/2201.11903).
* *7.5 The ML Model conforms to standards*: Yes (reuse of state-of-the-art RAG).
* *7.6 Verification of verification procedures*: No. But [Automated Unit Test Improvement using Large Language Models at Meta](https://arxiv.org/abs/2402.09171) 

## Data quality learning assurance objectives
* *1 Nature of data (explicit definition of input variables)*: Textual with metadata (dates and categorical)
* *2 Ranges of data (minimum and maximum value, classes of categorical data)*: Can be achieved with a [`describe`](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.describe.html) or [`unique`](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.Index.unique.html#pandas.Index.unique).
* *3 Representativeness: distribution of data across MLCODD [numerical values distribution, flight or operating condition, ambient condition indicator (e.g., day temperature for a pressure sensor, day, or night indicator for an image, rain, or dust level, etc.)]*: by design the data is rather exhaustive, but missing other commercial standards that are FAA accepted (SAE, ASTM, MIL-SPEC, RTCA).
* *11 Criteria for data augmentation, if applicable*: For NLP, data aurgmentation relies on the use of synomics, back-translations, regeneration...
* *13 Criteria for data segregation (training, validation, and verification datasets)*: Not different from standard practives.
* *14 Assurance Level*: N/A (DAL E).
* *16 Timeliness (Time stamp)*: FAA regularly cancels its publications, so there are mechanism against data drifts.
* *17 Completeness (statistically significant data to cover input space and OOD as per System/subsystem Level Requirements)*: By design the data is rather exhaustive, but missing other commercial standards that are FAA accepted (SAE, ASTM, MIL-SPEC, RTCA).
* *18 Format of dataset files*: Pandas [`dataframe`](https://pandas.pydata.org/pandas-docs/stable/reference/frame.html#dataframe).
* *20 Monitoring or Recording tag(s)*: I don't understand, but should be ensured by US gov.

# [ARP6983](https://www.sae.org/standards/content/arp6983/) Criteria Review 5b
This compliance assessment is a projection (is it attainable?) and I'm doing it mostly for myself. RAG is made of two MLCs: vector database embedding and LLM.

## SYSTEM DOMAIN AND ML CONSTITUENT OBJECTIVES

* *5.1 The MLC development is defined in the context of the applicable system-layer development and/or assurance process*: Compliant.
* *5.2 The MLC is assessed using the applicable system safety process*: Compliant.
* *5.3 The system architecture is developed considering the characterization of the MLC ODD and the MLC behaviors resulting from applying the ML development lifecycle*: Compliant.
* *5.4 Performance requirements based on metrics including acceptance targets, necessary thresholds, and limits are established to satisfy the safety requirements allocated to an MLC*: Not compliant. LLM metrics can be very unsatisfactory due to the subjectivity of text.
* *5.5 The system layer defines the attributes of the operating environment to a level of detail that is necessary to support MLC ODD characterization*: Compliant. Training envirronment is defined in conda [`environment.yml`](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#activating-an-environment).
* *5.6 The MLC, when implemented in the system, is verified to perform as intended in its allocated operating environment*: Compliant.

## MLDL OBJECTIVES AND OUTPUTS

### Objectives for MLC Requirements Process

* *1 MLCRs are developed from the system/subsystem requirements*: Compliant if we consider RAG as being a COTS reused (COTS has to be well documented).
* *2 The ML Constituent architecture is developed into MLCRs from the system/subsystem requirements including the system/subsystem architecture*: Compliant.
* *3 The MLC ODD is Characterized from the system/subsystem requirements and derived requirements coming from the other MLDL processes*: Compliant.
* *4 DQRs are developed as part of the MLCRs from the system/subsystem requirements and the MLC ODD characterization*: Compliant.
* *5 Derived MLCRs, including derived DQRs, are defined and provided to the System/Subsystem processes, including the System Safety Assessment process*: Compliant.

### Objectives for ML Data Management Process

* *1 ML Data Requirements are developed from the ML Constituent requirements*: Compliant. Retrospective definition based on FAA's DRS API.
* *2 Derived ML Data requirements are defined and provided to system/subsystem processes, including the system safety assessment process*: Compliant.
* *3 Data sources are identified and selected*: Yes (DRS).
* *4 Data are collected and corresponding data processing code is developed*: Compliant.
* *5 Data are prepared and corresponding data processing code is developed*: Compliant.
* *6 Data are labelled*: Compliant.
* *7 Data processing description is developed*: Compliant.
* *8 Data are allocated to training and validation datasets*: Compliant.
* *9 Data are allocated to test dataset(s) including scenario-based dataset if needed*: Compliant.

### Objectives for ML Model Design Process

* *1 The ML Model architecture is developed from the ML Constituent requirements*: Compliant if we consider RAG as being a COTS reused (COTS has to be well documented).
* *2 ML Model requirements are developed from the ML Constituent requirements*: Compliant if we consider RAG as being a COTS reused (COTS has to be well documented).
* *3 Derived ML Model requirements are defined and provided to system/subsystem processes, including the system safety assessment process*: Compliant.
* *4 The ML Model is developed*: Compliant.
* *5 The ML Model Description is developed*: Compliant.

### Objectives for Validation of Outputs of MLC Requirements Process

* *1 MLCRs comply with system/subsystem requirements*: Compliant, could be defined retrospectively.
* *2 MLCRs are traceable to system/subsystem requirements or identified as derived MLCRs which are provided to system/subsystem processes including the system safety assessment process*: Not Compliant.
* *3 MLCRs are accurate and consistent. The objective is to ensure that each MLCRs  is accurate, unambiguous, and sufficiently detailed, and that MLCRs do not conflict with each other*: Not Compliant, possible overlap between vector database embedding and LLM.
* *4 MLCRs are verifiable*: Compliant.
* *5 MLCRs conform to standards*: Compliant.

### Objectives for Validation of ML Data Requirements Produced by ML Data Management Process

* *1 ML data requirements comply with MLCRs*: Compliant.
* *2 ML data requirements are either traceable to MLCRs or identified as derived ML data requirements which are provided to system/-subsystem processes including the system safety assessment process*: Compliant.
* *3 ML data requirements are accurate and consistent*: Compliant.
* *4 ML data requirements and ML Model requirements are consistent*: Compliant.
* *5 ML data requirements conform to standards*: Compliant.

### Objectives for Validation of ML Model Requirements Produced by ML Model Design Process

* *1 ML Model requirements comply with MLCRs*: Not Compliant because of LLM.
* *2 ML Model requirements are either traceable to MLCRs or identified as derived ML Model requirements which are provided to system/subsystem processes including the system safety assessment process*: .
* *3 ML Model requirements are accurate and consistent*: Not Compliant.
* *4 ML Model requirements and ML data requirements are consistent*: Compliant.
* *5 ML Model requirements conform to standards*: Compliant.

### Objectives for Verification of Output of ML Data Management Process

* *1 Identified and selected data sources comply to MLCRs including DQRs and if applicable ML data requirements*: Compliant.
* *2 Collected data comply with MLCRs including DQRs and if applicable ML data requirements*: Compliant.
* *3 Prepared data comply to MLCRs including DQRs and if applicable ML data requirements*: Compliant.
* *4 Labels of prepared data comply with MLCRs including DQRs and if applicable ML data requirements*: Compliant.
* *5 Labels of prepareddata conform to ML data design standards*: Compliant.
* *6 Data processing complies with MLCRs (including DQRs and MLC architecture) and if applicable ML data requirements*: Compliant.
* *7 Data processing description is traceable to ML data processing*: Compliant.
* *8 Data processing description conforms to ML data design standards*: Compliant.
* *9 Training and validation datasets comply with ML data requirements*: Compliant.
* *10 Test datasets comply with MLCRs including applicable DQRs and if applicable ML data requirements*: Compliant.
* *11 Training and validation datasets are traceable to ML data requirements*: Compliant.
* *12 Test datasets and other ML verification artefacts are traceable to MLCRs including DQRs and if applicable ML data requirements*: Compliant.

### Objectives for Verification of Outputs of ML Model Design Process

* *1 ML Model architecture is compatible with MLCRs and if applicable with ML data requirements and ML Model requirements*: Not Compliant (LLM, vector embedding).
* *2 ML Model architecture is consistent*: Not Compliant.
* *3 ML Model architecture conforms to ML Model design standards*: Compliant.
* *4 ML Model complies with MLCRs (including MLC architecture) and if applicable ML Model requirements*: Not Compliant.
* *5 ML Model complies with ML Model architecture*: Compliant.
* *6 ML Model conforms to ML Model design standards*: Compliant.
* *7 ML Model (elements) is (are) consistent*: Compliant.
* *8 ML Model bias and variance are optimized*: Compliant.
* *9 ML Model generalization on in-sample data complies with MLCRs*: Compliant.
* *10 ML Model generalization on out-of-sample data complies with ML Model requirements*: Compliant.
* *11 ML Model is stable with MLCRs and if applicable ML Model requirements*: Not Compliant.
* *12 ML Model is robust with MLCRs and if applicable ML Model requirements*: Not Compliant.
* *13 ML Model description is traceable to ML Model*: Compliant.
* *14 ML Model description and ML data processing description are consistent*: Compliant.
* *15 ML Model description conforms to ML Model design standards*: Compliant.
* *16 ML Constituent is robust to outliers*: Not Compliant.
* *17 ML Constituent is robust to novelties*: Compliant.
* *18 ML Constituent is robust to data, concept, and performance drifts*: Compliant.

### Objectives for Verification of Results of ML Verification Process

* *1 Test procedures are correct*: Compliant.
* *2 Test procedures are traceable to test cases*: Compliant.
* *3 Verification results are correct, and discrepancies are explained*: Compliant.
* *4 Test results are traceable to test procedures*: Compliant.
* *5 Verification coverage of MLCRs is achieved*: Not Compliant.
* *6 Verification coverage of ML data requirements is achieved*: Not Compliant.
* *7 Verification coverage of ML Model requirements is achieved*: Not Compliant.
* *8 Verification coverage of MLC/MLM ODD is achieved*: Compliant.
* *9 ML Model explainability analysis is achieved*: Not Compliant.

# Risk-based levelling of objectives from [EASA Artificial Intelligence Concept Paper Issue 2 Guidance for Level 1 & 2 machine-learning applications](https://www.easa.europa.eu/en/downloads/139504/en)

## Trustworthiness analysis
* *ET-01 The  applicant  should  perform  an  ethics-based trustworthiness assessment for any AI-based system developed using ML techniques or incorporating ML models*: .
* *ET-02 The applicant should ensure that  the  AI-based system bears no risk of creating over-reliance, attachment, stimulating addictive behaviour, or manipulating the end user’s behaviour*: .
* *ET-03 The applicant should comply with national and EU data protection regulations (e.g. GDPR), i.e. involve their Data Protection Officer (DPO), consult with their National Data Protection Authority, etc*: .
* *ET-04 The applicant should ensure that the creation or reinforcement unfair bias in the AI-based system, regarding both the data sets and the trained models, is avoided, as far as such unfair bias could have a negative impact on performance and safety*: .
* *ET-05 The applicant should ensure that end users are made aware of the fact that they interact with an AI- based system, and, if applicable, whether some personal data is recorded by the system*: .
* *ET-06 The applicant should perform an environmental impact analysis, identifying and assessing potential negative impacts of the AI-based system on the environment and human health throughout its life cycle (development, deployment, use, end of life), and define measures to reduce or mitigate these impacts*: .
* *ET-07 The applicant should identify the need for new competencies for users and end users to interact with and operate the AI-based system, and mitigate possible training gaps*: .
* *ET-08 The applicant should perform an assessment of the risk of de-skilling of the users and end users and mitigate the identified risk through a training needs analysis and a consequent training activity*: .


## AI Assurance
* *DA-01 The applicant should describe the proposed learning assurance process, taking into account each of the steps described in Sections C.3.1.2 to C.3.1.14, as well as the interface and compatibility with development assurance processes*: .
* *DA-02 Based on (sub)system requirements that have been allocated to the AI/ML constituent, the applicant should capture the following minimum requirements for the AI/ML constituent: — safety requirements allocated to the AI/ML constituent; — information security requirements allocated to the AI/ML constituent; — functional requirements allocated to the AI/ML constituent; — operational requirements allocated to the AI/ML constituent, including AI/ML constituent ODD monitoring and performance monitoring, detection of OoD input data and data-recording requirements;
— other non-functional requirements allocated to the AI/ML constituent; and interface requirements*: .
* *DA-03 The applicant should define the set of parameters pertaining to the AI/ML constituent ODD, and trace them to the corresponding parameters pertaining to the OD when applicable*: .
* *DA-04 The applicant should capture the DQRs for all data required for training, testing and verification of the AI/ML constituent, including but not limited to: […]*
* *DA-05 The applicant should capture the requirements on data to be pre-processed and engineered for the inference model in development and for the operations*: .
* *DA-06 The applicant should describe a preliminary AI/ML constituent architecture, to serve as reference for related safety (support) assessment and learning assurance objectives*: .
* *DA-07 The applicant should validate each of the requirements captured under Objectives DA-02, DA-03, DA-04, DA-05 and the architecture captured under Objective DA-06*: .
* *DA-08 The applicant should document evidence that all derived requirements have been provided to the (sub)system processes, including the safety (support) assessment*: .
* *DA-09 The applicant should document evidence of the validation of the derived requirements, and of the determination of any impact on the safety (support) assessment and (sub)system requirements*: .
* *DA-10 Each of the captured (sub)system requirements allocated to the AI/ML constituent should be verified*: .

* *DM-01 The applicant should identify data sources and collect data in accordance with the defined ODD, while ensuring satisfaction of the defined DQRs, to drive the selection of the training, validation and test data sets*: .
* *DM-02-SL Once data sources are collected, the applicant should ensure that the annotated or labelled data in the data set satisfies the DQRs captured under Obj. DA-04*: .
* *DM-02-UL Once data sources are collected and the test data set labelled, the applicant should ensure that the annotated or labelled data in this test data set satisfies the DQRs captured under Objective DA-04*: .
* *DM-03 The applicant should define the data preparation operations to properly address the captured requirements (including DQRs)*: .
* *DM-04 The applicant should define and document pre- processing operations on the collected data in preparation of the model training*: .: .
* *DM-05 When applicable, the applicant should define and document the transformations to the pre-processed data from the specified input space into features which are effective for the performance of the selected learning algorithm*: .
* *DM-06 The applicant should distribute the data into three separate data sets which meet the specified DQRs in terms of independence (as per Objective DA-04): —  the training data set and validation data set, used during the model training; —   the test data set used during the learning process verification, and the inference model verification*: .
* *DM-07 The applicant should ensure validation and verification of the data, as appropriate, throughout the data management process so that the data management requirements (including the DQRs) are addressed*: .
* *DM-08 The applicant should perform a data verification step to confirm the appropriateness of the defined ODD and of the data sets used for the training, validation and verification of the ML model*: .

* *LM-01  The  applicant  should  describe  the  ML  model architecture*: .
* *LM-02 The applicant should capture the requirements pertaining to the learning management and training processes, including but not limited to: —   model family and model selection; —   learning algorithm(s) selection; —   explainability capabilities of the selected model; —   activation functions; —   cost/loss function selection describing the link to the performance metrics; —   model  bias  and  variance  metrics  and  acceptable levels; —   model    robustness    and    stability    metrics    and acceptable levels; —   training   environment   (hardware   and   software) identification; —   model parameters initialisation strategy; —   hyper-parameters identification and setting; —   expected performance with training, validation and test sets*: .
* *LM-03 The applicant should document the credit sought from the training environment and qualify the environment accordingly*: .
* *LM-04   The   applicant   should   provide   quantifiable generalisation bounds*: .
* *LM-05 The applicant should document the result of the model training*: .
* *LM-06 The applicant should document any model optimisation that may affect the model behaviour (e.g. pruning, quantisation) and assess their impact on the model behaviour or performance*: .
* *LM-07-SL The applicant should account for the bias- variance trade-off in the model family selection and should provide evidence of the reproducibility of the model training process*: .
* *LM-08 The applicant should ensure that the estimated bias and variance of the selected model meet the associated learning process management requirements*: .
* *LM-09 The applicant should perform an evaluation of the performance of the trained model based on the test data set and document the result of the model verification*: .
* *LM-10  The  applicant  should  perform  requirements- based verification of the trained model behaviour*: .
* *LM-11 The applicant should provide an analysis on the stability of the learning algorithms*: .
* *LM-12 The applicant should perform and document the verification of the stability of the trained model, covering the whole AI/ML constituent ODD*: .
* *LM-13 The applicant should perform and document the verification of the robustness of the trained model in adverse conditions*: .
* *LM-14  The  applicant  should  verify  the  anticipated generalisation bounds using the test data set*: .
* *LM-15 the applicant should capture the description of the resulting ML model*: .
* *LM-16 The applicant should confirm  that the trained model verification activities are complete*: .

* *IMP-01 The applicant should capture the requirements pertaining to the ML model implementation process*: .
* *IMP-02 The applicant should validate the model description captured under Objective LM-15 as well as each of the requirements captured under Objective IMP- 01*: .
* *IMP-03 The applicant should document evidence that all derived requirements generated through the model implementation process have been provided to the (sub)system processes, including the safety (support) assessment*: .
* *IMP-04 Any post-training model transformation (conversion, optimisation) should be identified and validated for its impact on the model behaviour and performance, and the environment (i.e. software tools and hardware) necessary to perform model transformation should be identified*: .
* *IMP-05 The applicant should plan and execute appropriate development assurance processes to develop the  inference model into software and/or hardware items*: .
* *IMP-06 The applicant should verify that any transformation (conversion, optimisation, inference model development) performed during the  trained model implementation step has not adversely altered the defined model properties*: .
* *IMP-07 The differences between the software and hardware of the platform used for training and those used for verification should be identified and assessed for their possible impact on the inference model behaviour and performance*: .
* *IMP-08 The applicant should perform an evaluation of the performance of the inference model based on the test data set and document the result of the model verification*: .
* *IMP-09 The applicant should perform and document the verification of the stability of the inference model*: .
* *IMP-10 The applicant should perform and document the verification of the robustness of the inference model in adverse conditions*: .
* *IMP-11 The applicant should perform requirements- based verification of the inference model behaviour when integrated into the AI/ML constituent*: .
* *IMP-12 The applicant should  confirm  that  the AI/ML constituent verification activities are complete.

* *CM-01 The applicant should apply all configuration management principles to the AI/ML constituent life- cycle data, including but not limited to: —   identification  of  configuration   items;  versioning; baselining; —   change control; reproducibility; —   problem reporting; —   archiving and retrieval, and retention period*: .

* *QA-01 The applicant should ensure that quality/process assurance principles are applied to the development of the AI-based system, with the required independence level*: .

* *RU-01 The applicant should perform an impact assessment of the reuse of a trained ML model before incorporating the model into an AI/ML constituent. The impact assessment should consider: […]*
* *RU-02 The applicant should perform a functional analysis of the COTS ML model to confirm its adequacy to the requirements and architecture of the AI/ML constituent*: .
* *RU-03 The applicant should perform an analysis of the unused functions of the COTS ML model, and prepare the deactivation of these unused functions*: .

* *SU-01 The applicant should capture the accuracy and fidelity of the reference model, in order to support the verification of the accuracy of the surrogate model*: .
* *SU-02 The applicant should identify, document and mitigate the additional sources of uncertainties linked with the use of a surrogate model*: .

* *EXP-01 The applicant should identify the list of stakeholders, other than end users, that need explainability of the AI-based system at any stage of its life cycle, together with their roles, their responsibilities and their expected expertise (including assumptions made on the level of training, qualification and skills)*: .
* *EXP-02 For each of these stakeholders (or groups of stakeholders), the applicant should characterise  the need for explainability to be provided, which is necessary to support the development and learning assurance processes*: .
* *EXP-03 The applicant should identify and document the methods at AI/ML item and/or output level satisfying the specified AI explainability needs*: .
* *EXP-04 The applicant should design the AI-based system with the ability to deliver an indication of the level of confidence in the AI/ML constituent output, based on actual measurements or on quantification of the level of uncertainty*: .
* *EXP-05 The applicant should design the AI-based system with the ability to monitor that its inputs are within the specified operational boundaries (both in terms of input parameter range and distribution) in which the AI/ML constituent performance is guaranteed*: .
* *EXP-06 The applicant should design the AI-based system with the ability to monitor that its outputs are within the specified operational performance boundaries*: .
* *EXP-07 The applicant should design the AI-based system with the ability to monitor that the AI/ML constituent outputs (per Objective EXP-04) are within the specified operational level of confidence*: .
* *EXP-08 The applicant should ensure that the output of the specified monitoring per the previous three objectives are in the list of data to be recorded per MOC EXP-09-2*: .
* *EXP-09 The applicant should provide the means to record operational data that is necessary to explain, post operations, the behaviour of the AI-based system and its interactions with the end user, as well as the means to retrieve this data*: .


## Human factors for AI
* *EXP-10 For each output of the AI-based system relevant to task(s) (per Objective CO-02), the applicant should characterise the need for explainability*: .
* *EXP-11 The applicant should ensure that the AI-based system presents explanations to the end user in a clear and unambiguous form*: .
* *EXP-12 The applicant should define relevant explainability so that the receiver of the information can use the explanation to assess the appropriateness of the decision / action as expected*: .
* *EXP-13 The applicant should define the level of abstraction of the explanations, taking into account the characteristics of the task, the situation, the level of expertise of the end user and the general trust given to the system*: .
* *EXP-14 Where a customisation capability is available, the end user should be able to customise the level of abstraction as part of the operational explainability*: .
* *EXP-15 The applicant should define the timing when the explainability will be available to the end user taking into account the time criticality of the situation, the needs of the end user, and the operational impact*: .
* *EXP-16 The applicant should design the AI-based system so as to enable the end user to get upon request explanation or additional details on the explanation when needed*: .
* *EXP-17 For each output relevant to the task(s), the applicant should ensure the validity of the specified explanation*: .
* *EXP-18 The training and instructions available for the end user should include procedures for handling possible outputs of the ODD monitoring and output confidence monitoring*: .
* *EXP-19 Information concerning unsafe AI-based system operating conditions should be provided to the end user to enable them to take appropriate corrective action in a timely manner*: .

* *HF-01 The applicant should design the AI-based system with the ability to build its own individual situation representation*: .
* *HF-02 The applicant should design the AI-based system with the ability to reinforce the end-user individual situation awareness*: .
* *HF-03 The applicant should design the AI-based system with the ability to enable and support a shared situation awareness*: .
* *HF-04 If a decision is taken by the AI-based system that requires validation based on procedures, the applicant should design the AI-based system with the ability to request a cross-check validation from the end user*: .
* *HF-05 For complex situations under normal operations, the applicant should design the AI-based system with the ability to identify a suboptimal strategy and propose an improved solution. Corollary objective: The applicant should design the AI- based system with the ability to process and act upon a proposal rejection from the end user*: .
* *HF-06 For complex situations under abnormal operations, the applicant should design the AI-based system with the ability to identify the problem, share the diagnosis including the root cause, the resolution strategy and the anticipated operational consequences. Corollary objective: The applicant should design the AI- based system with the ability to process and act upon arguments shared by the end user*: .
* *HF-07 The applicant should design the AI-based system with the ability to detect poor decision-making by the end user in a time-critical situation, alert and assist the end user*: .
* *HF-08 The applicant should design the AI-based system with the ability to propose alternative solutions and support its positions*: .
* *HF-09 The applicant should design the AI-based system with the ability to modify and/or accept the modification of task allocation pattern (instantaneous/short-term)*: .
* *HF-10 If spoken natural language is used, the applicant should design the AI-based system with the ability to process end-user requests, responses and reactions, and provide indication of acknowledged user’s intentions*: .
* *HF-11 If spoken natural language is used, the applicant should design the AI-based system with the ability to notify the end user that he or she possibly misunderstood the information*: .
* *HF-12 If spoken natural language is used, the applicant should design the AI-based system with the ability to identify through the end-user responses or his or her action that there was a possible misinterpretation from the end user*: .
* *HF-13 In case of confirmed misunderstanding or misinterpretation of spoken natural language, the applicant should design the AI-based system with the ability to resolve the issue*: .
* *HF-14 If spoken natural language is used, the applicant should design the AI-based system with the ability to not interfere with other communications or activities at the end user’s side*: .
* *HF-15 If spoken natural language is used, the applicant should design the AI-based system with the ability to provide information regarding the associated AI-based system capabilities and limitations*: .
* *HF-16 If spoken procedural language is used, the applicant should design the syntax of the spoken procedural language so that it can be learned and applied easily by the end user*: .
* *HF-17 If gesture language is used, the applicant should design the gesture language syntax so that it is intuitively associated with the command that it is supposed to trigger*: .
* *HF-18 If gesture language is used, the applicant should design the AI-based system with the ability to disregard non-intentional gestures*: .
* *HF-19 If gesture language is used, the applicant should design the AI-based system with the ability to recognise the end-user intention*: .
* *HF-20 If gesture language is used, the applicant should design the AI-based system with the ability to acknowledge the end-user intention with appropriate feedback*: .
* *HF-21 In case spoken natural language is used, the applicant should design the AI-based system so that this modality can be deactivated for the benefit of other modalities*: .
* *HF-22 If spoken (natural or procedural) language is used, the applicant should design the AI-based system with the ability to assess the performance of the dialogue*: .
* *HF-23 If spoken (natural or procedural) language is used, the applicant should design the AI-based system with the ability to transition between spoken natural language and spoken procedural language, depending on the performance of the dialogue, the context of the situation and the characteristics of the task*: .
* *HF-24 The applicant should design the AI-based system with the ability to combine or adapt the interaction modalities depending on the characteristics of the task, the operational event and/or the operational environment*: .
* *HF-25 The applicant should design the AI-based system with the ability to automatically adapt the modality of interactions to the end-user states, the  situation, the context and/or the perceived end user’s preferences*: .
* *HF-26 The applicant should design the AI-based system to  minimise  the  likelihood  of  design-related  end-user errors*: .
* *HF-27 The applicant should design the AI-based system to minimise the likelihood of HAIRM-related errors*: .
* *HF-28 The applicant should design the AI-based system to be tolerant to end-user errors*: .
* *HF-29 The applicant should design the AI-based system so that in case the end user make an error while interacting with AI-based system, the opportunities exist to detect the error*: .
* *HF-30 The applicant should design the AI-based system so that once an error is detected, the AI-based system should provide efficient means to inform the end user*: .


## Safety risk mitigation
* *SRM-01 Once activities associated with all other building blocks are defined, the applicant should determine whether the coverage of the objectives associated with the explainability and learning assurance building blocks is sufficient  or  if an additional dedicated layer  of protection, called hereafter safety risk mitigation, would be necessary to mitigate the residual risks to an acceptable level*: .
* *SRM-02 The applicant should establish safety risk mitigation means as identified in Objective SRM-01*: .


## Additional risk-based levelling of information-security-related objectives
* *IS-01 For each AI-based system and its data sets, the applicant should identify those information security risks with an impact on safety, identifying and addressing specific threats introduced by AI/ML usage*: .
* *IS-02 The applicant should document a mitigation approach to address the identified AI/ML-specific security risk*: .
* *IS-03 The applicant should validate and verify the effectiveness of the security controls introduced to mitigate the identified AI/ML-specific security risks to an acceptable level*: .

