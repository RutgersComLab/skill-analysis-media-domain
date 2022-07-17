# Identifying trends in the the US media industry

## Overview
In this project, we try to identify how the skill requirements have changed within the US media domain in the past decade. We use the USA Labor Insight dataset provided by (Lightcast)[https://www.economicmodeling.com/] (formerly known as Burning Glass Technologies) to perform the analysis. The dataset is a text dump of 5 entities - main, certifications, cip, degree, major, and skills - amounting to more than 100 GB in size. It holds data from 2010 to 2021 as well as data from 2007, and each record holds information about a particular job posting. We use [PySpark](https://spark.apache.org/docs/latest/api/python/) to process and save the data as [Parquet](https://parquet.apache.org/), before loading it into [Amazon Redshift](https://aws.amazon.com/redshift/). We also employ the use of different techniques and packages such as [Sentence Transformers](https://github.com/UKPLab/sentence-transformers) and [Clustering](https://scikit-learn.org/stable/modules/clustering.html) along with other popular python packages to find insights in the data.

## Problem Statements
The objective is to identify how the skill requirements for different jobs in the US media domain has changed in the past decade. We try to get a sense of which skills were discarded and what new skills were adopted for paricular occupation domains. We concentrate on the following occupation groups:

| ONET Code | Occupation Group  |
| :-----: | :-: |
| 27-* | Media and Communication Workers |
| 13-* | Business Operations Specialists |
| 15-* | Computer Occupations |
| 41-* | Sales |

We also run a sub-analysis for news media by loooking at the trends for the following ONET codes:
| ONET Code | Occupation Group  |
| :-----: | :-: |
| 27-4021.00 | Photographers |
| 27-3043.05 | Poets, Lyricists and Creative Writers |
| 27-3011.00 | Broadcast Announcers and Radio Disc Jockeys |
| 25-1122.00 | Communications Teachers, Postsecondary |
| 27-3041.00 | Editors |
| 27-4032.00 | Film and Video Editors |
| 43-9081.00 | Proofreaders and Copy Markers |
| 27-4012.00 | Broadcast Technicians |
| 27-3042.00 | Technical Writers |
| 27-2012.03 | Media Programming Directors |
| 27-2012.05 | Media Technical Directors/Managers |

## Implementation
For our analsyis, we use the main and skills tables, with data ranging from 2010 to 2021. The main table holds the core details regarding the job posting such as title, onet, employer, sector, naics, location, education, degree, experience, salary_type etc. Skills table holds information about what skills are required for each job. The implementation consists of the following stages:
<ol>
	<li>Load the raw text files into a relational DB.</li>
	<li>Identify filters that gives the data realted to 'Journalism and Media' category.</li>
	<ul>
		<li>The relevant filters were identified as NAICS, ONET, and Year columns.</li>
	</ul>
	<li>Identify job skill clusters using Cluster analysis for each identified ONET code.</li>
	<ul>
		<li>Using Word2Vec to create the dense vector representations (did not work).</li>
		<li>Using Sentence Transformers to create the dense vector representations.</li>
	</ul>
	<li>Build comparative plots to help visualize how the job skill clusters change by year and occupation group.</li>
	<li>Perform a 'sub-analysis' for the News media category in particular.</li>
</ol>
Some of these stages are discussed below:

### Loading the raw data to a relational DB

Skills             |  Main
:-------------------------:|:-------------------------:
![Skills_raw file](data/raw/skills.png?raw=true "Skills_raw file")  |  !![Main_raw file](data/raw/main.png?raw=true "Main_raw file")




### Clustering the skills

### Visualization
We also build a visualization on top of the results.

## Results
In this section, we will discuss the results of the analysis:
***TBD***

## Packages/References/Credits
Item | Link
--- | ---
LDA | https://radimrehurek.com/gensim/models/ldamodel.html
LDASeq | https://radimrehurek.com/gensim/models/ldaseqmodel.html
BERTopic | https://github.com/MaartenGr/BERTopic/
BERTopic-DTM | https://github.com/MaartenGr/BERTopic/#dynamic-topic-modeling
NLTK | https://www.nltk.org/
Gensim | https://radimrehurek.com/gensim/
spaCy | https://spacy.io/
Plotly | https://plotly.com/
Pandas | https://pandas.pydata.org/
Numpy | https://numpy.org/
Haystack | https://haystack.deepset.ai/overview/intro
scikit-learn | https://scikit-learn.org/stable/index.html
pyLDAvis | https://github.com/bmabey/pyLDAvis
The New York Times | https://www.nytimes.com/
ProQuest | https://www.proquest.com/

[lda]:https://radimrehurek.com/gensim/models/ldamodel.html
[lda_Seq]:https://radimrehurek.com/gensim/models/ldaseqmodel.html
[bertopic]:https://github.com/MaartenGr/BERTopic/
[bertopic_dtm]:https://github.com/MaartenGr/BERTopic/#dynamic-topic-modeling
[nltk]:https://www.nltk.org/
[gensim]:https://radimrehurek.com/gensim/
[spacy]:https://spacy.io/
[plotly]:https://plotly.com/
[pandas]:https://pandas.pydata.org/
[numpy]:https://numpy.org/
[haystack]:https://haystack.deepset.ai/overview/intro
[scikit_learn]:https://scikit-learn.org/stable/index.html
[pyldavis]:https://github.com/bmabey/pyLDAvis
[nyt]:https://www.nytimes.com/
[proquest]:https://www.proquest.com/