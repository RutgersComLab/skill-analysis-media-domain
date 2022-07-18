# Identifying Trends in the the US media industry

## Overview
In this project, we try to identify how the skill requirements have changed within the US media domain in the past decade. We use the USA Labor Insight dataset provided by [Lightcast](https://www.economicmodeling.com/) (formerly known as Burning Glass Technologies) to perform the analysis. The dataset is a text dump of 5 entities - main, certifications, cip, degree, major, and skills - amounting to more than 100 GB in size. It holds data from 2010 to 2021 as well as data from 2007, and each record holds information about a particular job posting. We use [PySpark](https://spark.apache.org/docs/latest/api/python/) to process and save the data as [Parquet](https://parquet.apache.org/), before loading it into [Amazon Redshift](https://aws.amazon.com/redshift/). We also employ the use of different techniques and packages such as [Sentence Transformers](https://github.com/UKPLab/sentence-transformers) and [Clustering](https://scikit-learn.org/stable/modules/clustering.html) along with other popular python packages to find insights in the data.

## Objective
The objective is to identify how the skill requirements for different jobs in the US media domain has changed in the past decade. We try to get a sense of which skills were discarded and what new skills were adopted for particular occupation domains. We concentrate on the following occupation groups:

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
For our analsyis, we use the main and skills tables, with data ranging from 2010 to 2021. The main table holds the core details regarding the job posting such as title, onet, employer, sector, naics, location, education, degree, experience, salary type etc. Skills table holds information about what skills are required for each job. Both main and skills exists in normalized from. The implementation consists of the following stages:
<ol>
	<li>Load the raw text files into a relational DB (Amazon Redshift).</li>
	<li>Identify filters that gives the data realted to 'Journalism and Media' category.</li>
	<ul>
		<li>Create two denormalized tables using main and skills, as well as main and certifications.</li>
		<li>From these, we identify the relevant filters as NAICS, ONET, and Year columns.</li>
	</ul>
	<li>Identify job skill clusters using cluster analysis for each identified ONET code.</li>
	<ul>
		<li>Using Word2Vec to create the dense vector representations (did not work).</li>
		<li>Using Sentence Transformers to create the dense vector representations.</li>
	</ul>
	<li>Build comparative plots to help visualize how the job skill clusters change by year and occupation group.</li>
	<li>Perform a 'sub-analysis' for the News media category in particular.</li>
</ol>
Some of these stages are discussed below:

### Loading the Raw data into a Relational DB
The raw data is available as text files and the has the format shown below:

Skills             |  Main
:-------------------------:|:-------------------------:
![Skills_raw file](screenshots/skills.png?raw=true "Skills_raw file")  |  ![Main_raw file](screenshots/main.png?raw=true "Main_raw file")

We use [PySpark](https://spark.apache.org/docs/latest/api/python/) to load the data and perfrom basic EDA on them. This gives us information such as record count, skewness in the data, how the tables are normalized, what columns are available etc. We also create a few columns like day, month, and year. Finally, we partition the data by year and save it in [Parquet](https://parquet.apache.org/) file format. We use Parquet as it is optimized to work with complex data in bulk and uses one of the best compression techniques available. It also minimizes the IO by reading only the required columns and partitions. Redshift makes it easy to create external tables on top of Paarquet, thereby making it an ideal candidate. A screenshot of the final raw layer is given below:

![Raw Layer](screenshots/raw%20layer.png?raw=true "Raw Layer")

### Clustering Skills
Once we have identified the subset of data to use for the anlaysis, the next step is to create job skill clusters and understand how they have evolved over time. A combination of year, ONET, and NAICS gives us the right subset of the data to perfrom the analysis on. Since we want to represent occupation domains, we do the clustering on an ONET level. The clustering itself can be broadly split into two parts:

#### 1. Getting data in the right format
We get the top 25 records for each year and onet combination using the below query:
```
select onet, skill, count(skill) as count from jobrole.main_skill_cleansed 
WHERE naics_6 IN ('519130', '511110', '518210', '511199') 
  AND year=<YEAR>
  AND skill!='na' 
  AND onet LIKE <ONET CODE>
group by onet, skill 
ORDER BY count DESC
limit 25
```
The full query, as well as the output can be viewed [here](data/Top%2025%20skills%20\(By%20Year%20and%20ONET%20_%20with%20NAICS%20filter\).xlsx).

#### 2. Cluster Analysis
We can think about this as having two components - Clustering and Cluster Evolution. Clustering can be thought of as plotting the skills into an n-dimensional space, and Cluster Evolution helps in identifying how these clusters have changed over the years. The steps involved in Clustering are as follows:
<ol>
	<li>Get all the skills related to a single ONET code across all years. For example, 'get all the skills associated with ONET 15 from 2010 to 2021'.</li>
	<li>Convert the skills into dense vector embeddings. Dense vectors are numerical representations of semantic meaning. Words are encoded into high-dimensional vectors, and the abstract meaning and relationship of words are numerically encoded.</li>
	<ul>
		<li>The first approach was using Word2Vec pre-trained models. Both of these failed to create the embeddings as some of the skills were out of vocabulary (OOV) for those models. Some of the examples for the OOV words were 'Web Development' and 'Microsoft Excel'. The models tried were 'conceptnet-numberbatch-17-06-300' and 'word2vec-google-news-300', which are two of the biggest provided by Gensim.</li>
		<li>The second approach was using Sentence Transformers to create the dense vector representations. This works as expected and we are able to represent all the skills as vector embeddings.</li>
	</ul>
	<li>Once the embeddings have been created, we use k-means clustering to train a model. This model helps in mapping the embeddings onto a space, and identify clusters in them. We calculate the Silhouette Score to get a sense of how good the clusters generated are.</li>
	<li>Save the clusters as a JSON file for future reference.</li>
</ol>
Repeat the following steps for each year to perform Cluster Evolution:
<ol>
	<li>Get all the skills related to a single ONET code in a particular year. For example, 'get all the skills associated with ONET 15 for the year 2010'.</li>
	<li>Assign each skill to the space we created earlier by predicting its closest cluster.</li>
	<li>For each cluster, get the total number of times all the skills part of the cluster was asked for. This gives us a measure of how demanding a particular skill cluster is in a particular year.</li>
</ol>

Repeat the above steps for each ONET we want to analyze. The code for this section is available [here](code/Skill%20Trend%20Analysis.ipynb), and results can be viewed [here](https://github.com/jacobceles/skill-analysis-media-domain/tree/main/results/).

### Visualization
We build 3 different visualizations on top of the results:
<ol>
	<li>Bubble chart - A bubble chart representation of how each cluster has changed across the years. The bubbles represent a cluster and the size is a measure of how interesting it was for that year.</li>
	<li>Line chart - To be used along with the cluster data saved as json. This plot is a line chart showing how each cluster has changed across the years. Can be filtered by double-clicking the legend to the right. You can get more information by hovering over each point.</li>
	<li>Bar chart - A plot consisting of 10 subplots (2010-2021), each showing the frequency of clusters within a year. Can be used to identify which cluster was the most important each year.</li>
</ol>

All these visualizations can be viewed [here](https://github.com/jacobceles/skill-analysis-media-domain/tree/main/results/)

## Results
In this section, we will discuss the results of the analysis:
***TBD***

## Packages/References/Credits
Item | Link
--- | ---
Lightcast (formerly known as Burning Glass Technologies) | https://www.economicmodeling.com/
Sentence Transformers | https://github.com/UKPLab/sentence-transformers
Gensim Models | https://github.com/RaRe-Technologies/gensim-data#models
Amazon Redshift | https://aws.amazon.com/redshift/
matplotlib | https://matplotlib.org/
Plotly | https://plotly.com/
Pandas | https://pandas.pydata.org/
Numpy | https://numpy.org/
scikit-learn | https://scikit-learn.org/stable/index.html
PySpark | https://spark.apache.org/docs/latest/api/python/
Parquet | https://parquet.apache.org/