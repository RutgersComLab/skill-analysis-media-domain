# Identifying trends in the the US media industry
data provided by 
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