# A NLP project on FAA documents

## Abstract
This is a personal NLP project dealing with FAA documentation: use case definition, data source identification, data collection, data cleaning, data mining, RAG and LLM fine-tuning.

# Introduction

## Motivation
While aviation is a heavy producer of documentation, most of the aviation-related database publicly available on popular ML websites (kaggle, huggingface, google data) is about unformatted tabular data and not about NLP (e.g. accident rates, airport traffic, helicopter types). On the contrary legal has a several datasets available.
This is quite unusual consedering that aviation data is pretty standardized.
While it takes at least 10 years for a commercial-level technology to hit aviation, data has been at the core of aviation for a lot of time (CVFDR, telemetry...).
FAA has been offering for several years a moderm and powerful platform to browse all the publicly available FAA documentation: https://drs.faa.gov/. This platform offers tremendous capabilities and offers an API (one of the many DOT platforms to offer this) to programaticaly acccess its docuymentation.
Non governmetal platforms like AHST offer also high qulity documentation and formatter, but haev no API.

## Why aviation data
The starting block of any ML project is to get to intimately familiar with the data. Since I work in aviation, I've a professional interest in knowing more about all types of documents format and content that the FAA produces. The availability of a structured and rich dataset is also an enabler for integration of ML techniques.
From a pure ML perspective, aviation data offers lots of benefits:
* because of the safety constraints, aviation structuraly produces lots of data in various format 
* aviation data is diversifed (everywhere where there is an airport) and standardized (FAA Forms 8xxx-x, Tech logs, ATC communication, ARINC 429 GAMA, accident reports...)
* aviation data has been around for several decades
And more specifically for the text documents released by the FAA as a usage doma:
* FAA offers a powerful API
* FAA are well formatted (Q&A tables, consistent tables of content)
* FAA documents have a high quality standard (Order 1000.36) compared to the best NLP dataset that one can find on Hugging Face


## SAE Criteria
Being a member of SAE G-34 Artificial Intelligence in Aviation, I wanted to apply the pucklish to this porject:


Attempt | #1 | #2 | #3 | #4 | #5 | #6 | #7 | #8 | #9 | #10 | #11
--- | --- | --- | --- |--- |--- |--- |--- |--- |--- |--- |---
Seconds | 301 | 283 | 290 | 286 | 289 | 285 | 287 | 287 | 272 | 276 | 269

## Data Collection
### Results on data collection
Those are the steps that I followed for collecting data from DRS:
1. Get familiar with the API: https://drs.faa.gov/help/helpdetails
2. Download the DRS Document Types Metadata Mapping file from https://drs.faa.gov/help/helpdetails
3. Collect the index table for each DRS doc type
4. Collect each file or text from each document found in each index table
This is pretty straghtforward except for several hard points:
1. DRS website changed a little bit, so the code had to follow
2. Some files caused some issue (files with special characters like &)
3. The data scrapping portion relies on Selenium. After some times my IP address got banned, so I started using a VPN to finish my data collection
4. On top of the main file, DRS offers also "Attachments/Public Comments" documents
5. There also non-pdf and non-text files
Retrieving data from FAA DRS proved to be very convenient.

Note that Aeronautical Information Manual (AIM) and Aeronautical Information Publication (AIP) are two basic references in aviation. They are not in DRS, so I've added them manually to the dataset.

### Other sources of data worth exploring
The following data could also be an excellent source of data for an NLP project. I've not had the time to focus too much on those, but they would be well suited for a simple use case:
* public comments are organized as Q&A tables. There are generally very nicelly formatted so that a tabular extrator libraty should work (sometimes alswer formatted in csv like files). The address pretty much all the time complex questions (referrence to other FAA documents or industry practive). The answer provided by the FAA is not "right" like what can be found in other factual Q&A database, but the clarify the intent of a guidance.
* AC 43-16A - General Aviation Maintenance Alerts offers also an excelent source of maintenance database
* MMELs are also well formatted, but definitely less diversified
* FAA DER ressources for LLM (learning documents and Q&A)
* all the DRS missing data and all the documents which were discontinued to exist

## A first try with STC (Supplemental Type Certificate)

### Why STC certificates
STC is the master document that defines the type design of an aircract modification, it's the final document (except for the PMA) that gets issued at the end of a new design project. I happen to be working a lot with STCs through my work, so I knew that those enjoyed lots of nice properties:
* small (<1000 tokens)
* rigid structure
* informative
* diverse
* long history

### Metadata only
The DRS metadadata already offers a wealth of information before even starting exploring the text content. I've only analysed a fraction of the metadata and the following is another fraction of this analysis. It's really a high-level analysis to get to know the data better, but I didn't push towards more complex data mining methods (dimensiality reduction, clustering... scikit-learn is the best source I know to start a ML project). Maybe something for later.
There will be as many rows for an STC as there are issuance. Date of STC (re)-issuance is given by "stcStatusDate". A STC is reissued for example to add a new configuration (e.g. obsolescence). Note that an STC can be updated (e.g. minor change for editorial changed) and not reissues and in that case the lates update will be indicated in the field "docLastModifiedDate".
"drs:stcAplicationDate" corresponds to the application date for the firs STC issuance.

* Number of STC (documents):
  * indexed: 76416
  * with PDF: 43818
* Sanity check:
  * only one broken PDF (SA00765DE)
  * very few abnormal dates (<50)

* Rotorcraft STCs reissuance period: is about 3 years, but we can see actually that the period is more like 1.6 years and that ther is another peak around 7 yeahs (~1.7*4). It could be interesting to filter by ATA, company, ACO... to try to consolidate this patter. This other interesting observation is that is period is not consitent when we switch to airplanes.

![plot](./images/20_rotorcraft_reissuance.png)

* Number of STC re-issuance for rotorcrafts: the majority of STCs are not reissued.

![plot](./images/20_number_of_STC_issuances_for_rotorcraft.png)

* Number of current and historical STCs for the different FAA offices: it's quite expected to see Fort Worth on top of the list.

![plot](./images/20_Number_of_STCs_by_Office_all_years.png)

* Current STCs re-issuance for the different FAA offices since 2020: interrestingly Fort Worth is not on the top position while the group of other top FAA offices remains the same.

![plot](./images/20_Number_of_STCs_by_Office_2020_rotorcraft.png)

* Histogram of year of initial applications: I was surprise to observe that new rotorcraft STCs have only been decreasing since 2008 (US financial crisis, but it might be just a correlation).

![plot](./images/20_rotorcraft_stcApplicationDate_by_year.png)


* Histogram of year of STC reissuance: we can still see the decrease, but it's unclear what caused those 3 peaks without drilling more into the data.

![plot](./images/20_rotorcraft_stcStatusDate_by_year.png)

* Histogram of application day of the week: I would have expected a flat distribution, but it's interresting to see a peak at the beginning/end of the month. This might be driven by company business objectives. Harder to explain is the peak in the middle of the month.

![plot](./images/20_rotorcraft_stcStatusDate_by_day.png)

* Histogram of application day of month: The distribution is rather flat as expected, with a sharp drop on Friday (not surprising). Most surprising is that there are a few applications on Saturdays.

![plot](./images/20_rotorcraft_stcStatusDate_by_day_of_week.png)

* Histoirical helicopter STCs between 1990 and 2000: a big surprise to me was that American Eurocopter (my curent company and now Airbus Helicopters, Inc.) was one of the largest issuers of STC certificates (but a detailed analysis of the STCs show that those are not complex). Cheers to the team of engineers, because I'm walking on their shoulders everyday. It's also worth exploring noting that the different companies have very different business models (ODA doing worth for third parties, ODA doing only internal work, companies with core products working with ACO).
![plot](./images/20_STC_by_STC_holder_helicopter_1900.png)

* Historical and Current STCs: the distriution is quite expected, but most surprising is that the distribution is not reflecting the relative weight of those markets (large multi-engine should be significantly highier).

![plot](./images/20_STCs_by_stcProductSubType.png)

I've looked at more histograms and I could keep going on for many more pages (e.g. looking at time drifts, comparing single engine vs. multi-engines...), but that's enough histograms for today (and it's still a mandatory quality gate for a ML project).

### Adding text content
STC textual data was decoded using a mix of from pypdf and PyPDF2 for pure PDF decoding and google cloud platform OCR when only a scan was available.


oparler de la quantiti'e de fichier, noom,bre de mots,,,

While LLM now accept long token

I started working on ATA, but I dropped for good reasons.


This is a nice Gama distribution (as expected).

olllama

lmsutio

22

![plot](./images/22_limitations_vs_descriptions.png)
![plot](./images/22_number_of_words_in_description.png)
![plot](./images/22_number_of_words_in_limitations.png)



abnormal dates... just a dizen


* Number of STCs by stcProductSubType all years
* Number of STCs by stcProductSubType by decade
* Number of STCs by Holder all years
* Number of STCs by Office all years
FW behind now
* Number of STCs by Office by decade
lots of other comments possibnle, but stop here.
* STC dates
last modified date is very odd
Same but by decade
Delta time between reissuances
Number of reissuances


#### text content

no useful pattenrs. but it's normal. The two fielsds shsare little common data. There are some mandatory fields, very well formatted.


very few shared information


lots of very unique way to designate something eimislar. Eg radio called by P/M

variations on the meaning of waht's a limitation and hat's a definiftion

edecodin in google cloud platform

number os usefl (43818, 30)


pas grand chose

qques parrters = l;igne droite

interessant de' explorere les cas dans l' enveloppe


dire que c' est fait des essais avec LLM.

tuens out that it was not a good idea: why?

Reduced scope and subset of doc types 


General overview of DRS data
Getting FAA data from the DRS

Case study on STC

Some metrics


talk about the few problems in the dates

## DRS and the other non DRS documents

### Data collection

cite des use case

For the data .

FAA API, data and DRS

Is that suitable for nlp? Os that suitable for nlp on a personal computer?

Challenge with DRS data
What's not in FAA DRS
Getting to know the FAA DRS better

Parsibt Drs docs

Parsing non Drs docs

Is that suitable for NLP?


## analysis

I started writting a quick analysis of the documentary inflation, but I think it could be wronlgy interpreted. More documentation is better. If regulateors does not writte the rules, someone else will (end customers, customers) who are not "by design" concerned by safety.


## ML applications

I did seveal prompts (wit)

zero shot proved very innefective. I didn't try chatGPT.


fdire que je suis parti sur du RAG, car prometeur. Mais tarining sur la liste en utlisaaloxe (voir article de HN), amsi computer intesive.

j'ai fait des experienc avec LLM, mais il faut de lonuges tokes window et de la puiossance de calcul, genre 5k

apres le segmentage permet de reduide la puissance  calcul


prompt pour le llm:
qui tu es
contexte 


role: adapt role de' exper

task:  review against standard and rpviode advice

context: target audience, background information, scenario


# Acknowledgment



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