# Data_Systems_for_Analytics

[All Answers](../All_Answers.md)

## August 2022

**a) Select the correct statements below:**

(a) A well-defined relational database schema guarantees that all data in the database is correct. (−50%)

(b) Societal value is a valid reason to start a big data collection. (50%)

(c) Sequential reads are important for big data analytics applications. (50%)

(d) Big data collections only contain structured data. (−50%)

**b) Briefly discuss Velocity in big data applications.**

- Velocity typically refers to the fact that data is created at a fast pace, and we must manage this fast pace to capture the data for future use. It can also, however, refer to the situations where the data is most relevant if it can be acted upon quickly, e.g., in a fast-paced financial environment or in a sensoring application, where the censors detect a problem that must be resolved.

## Maj 2022

**a) Select the correct statements below:**

(a) At its core, cloud storage uses the same storage media as local storage: HDDs and/or SSDs. (33.33333%)

(b) Deep learning is often used to train classifiers that can translate unstructured multimedia data (images and video) to more structured text data. (33.33333%)

(c) Data Volume alone is not sufficient to consider a collection of data to be a big data collection. (33.33333%)

(d) Data is never written to storage in a big data collection. (−100%)

**b) Discuss the pros and cons of using Spark to process and manage big data collections.**

- Spark is excellent at processing big data collections, as it can work with distributed storage and run complex computations in a distributed manner. It is not really a data management system, however, as it simply processes data from a file system.

## December 2021

**a) Select the correct statements below:**

(a) All big data applications are evil in nature. (−50%)

(b) Spark is designed for supporting user-facing and interactive applications. (−50%)

(c) Many big data applications require so much computation that a distributed processing framework is necessary. (100%)

(d) A machine learning model that is based on real data can never have any bias, because it accurately models real life. (−50%)

**b) What is the the most important disk access pattern of typical big data applications? Support your answers with arguments.**

- The most important access pattern is sequential reads. Adding ‘‘ over large collections’’ would be even more accurate. This is because the typical big data applications aim to model the entirety of the data somehow, and to do that the whole data must be read. Furthermore, many approaches require repeatedly reading the collection. However, the specific order of reading the collection is rarely important, allowing for sequential reads in whatever order the collections happen to be in.

## March 2021

**7a) Select the true statements:**

(a) Without the ability to join relations, there would be no big data analytics applications. (0%)

(b) Value in big data analytics refers to more than monetary value. (50%)

(c) In big data applications, there is no need to worry about inserting new data. (0%)

(d) A major reason why Spark is more e cient than Hadoop is that it uses memory more effectively. (50%)

**7b) Write your reections here:**

- Since the clusters must be created incrementally, they must potentially be written in a random fashion. In each round of the algorithm, the data can therefore be read sequentially, but must be written randomly. Since SSDs improve random IOs relatively more than sequential IOs, the writing phases is improved more by SSDs.

## Januar 2021

**7a) Select the true statements:**

(a) Hadoop is significantly more efficient than Spark for complex processing pipelines. (0%)

(b) The main data access pattern of analytics systems consists of full sequential scans of large data collection. (33.33333%)

(c) Using machine learning on existing data is likely to produce models that have built-in bias. (33.33333%)

(d) In big data, Volume refers to the massive quantities of data that must be stored and processed. (33.33333%)

**7b) Write your reflections here:**

- Example answer: The main role of database systems is to store and deliver data, regardless of how that data is used. Hence, the role of database systems in eliminating bias is at best indirect. Database systems do have many tools for querying and analysing data, and hence could be used to detect and analyse bias, and so they can indeed have an indirect role to play.

## August 2020

**7a) Select the true statements:** (a) Data in big data collections is never correct. (0%)

(b) Clustering algorithms can be used to identify groups of related records in, for example, banking applications. (33.33333%)

(c) Key-value stores are better for web-caching than analytics applications. (33.33333%)

(d) Spark is significantly more flexible than Hadoop for complex processing pipelines. (33.33333%)

**7b) Write your reflections here:**

- Simply saying that the high latency of Spark prevents quickly answering queries would be worth some points. Explaining the reason why (text files have no index, so to find the answer to the query, the whole large text file must be read and processed, which is done using multiple servers with communication overhead) would complete the answer.

## April 2020

**a) Select the correct statements below:**

(a) Spark is more e cient than Hadoop for most Big Data applications. (50%)

(b) Videos are generally considered structured data. (0%)

(c) Spark has very low latency for small tasks. (0%)

(d) Distributed processing is important for most Big Data applications. (50%)

**b) Discuss the meaning of Variety in Big Data applications.**

- Variety means that the data can a) be from various origins, b) address various aspects of the big data archive, and c) have various formats.

## Maj 2020

a) Select the correct statements below:

(a) Big data is always correct data. (0%)

(b) Most machine learning algorithms will learn the biases present in the training data. (50%)

(c) Document stores are the best tool for big data analytics applications. (0%)

(d) Although Hadoop is fairly recent technology, the MapReduce concept is very old. (50%)

b) Contrast briefly the different types of Value in Big Data applications.

- We discussed two types of values: financial and societal. Most companies would desire financial value from their big data applications, for example in the form of improved operations and profits. Governmental and scientific institutions are more likely to go for societal value, for example in the form of improved public health or scientific knowledge. Of course, these may mix in arbitrary proportions, and societal value often results in financial value.
