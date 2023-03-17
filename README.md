# Amazon_Vine_Analysis
Amazon Web Services and Cloud Based Data Extraction and Analysis of Amazon Product Customer Reviews

## Overview
Big Data was extracted from an Amazon Web Services (AWS) server and processed with PySpark (Spark in Python) in a Google Colab .ipynb file.
The data was then transferred to a postgres SQL server in pgAdmin. This SQL server was hosted by AWS.

### Purpose

The purpose of this analysis was to extract Amazon customer review data (Musical Instruments dataset) from an AWS server into a Cloud based computing program called Google Colab. We used PySpark in Google Colab to read a csv from the AWS server and process the resulting dataframe into four data tables that matched the schema of a Postgres database. Ultimately, we facilitated the transfer of data from an AWS server to a postgres database hosted by our own personal AWS server.

I performed these "Big Data" operations to assess whether or not paid customer reviews on Amazon were biased.
To evaluate the bias, I determined the percentage of paid reviews and unpaid reviews that had a a five-star rating.
Evidence for customer review bias would be supported by a higher five-star rating for paid customer reviews compared to unpaid customer reviews.

## Analysis
I selected the musical instruments dataset from Amazon. 
To begin, a Postgres driver was imported into Google Colab.
`!wget https://jdbc.postgresql.org/download/postgresql-42.2.16.jar`

A spark session was created and the Postgres driver was loaded into Spark.

`from pyspark.sql import SparkSession`

`spark = SparkSession.builder.appName("M17-Amazon-Challenge").config("spark.driver.extraClassPath","/content/postgresql-42.2.16.jar").getOrCreate()`

The musical instruments dataset was then extracted from a proprietary AWS server (different from the server that was used to host our Postgres database) and was loaded into Google Colab using Spark.

`from pyspark import SparkFiles`

`url = "https://s3.amazonaws.com/amazon-reviews-pds/tsv/amazon_reviews_us_Musical_Instruments_v1_00.tsv.gz"`

`spark.sparkContext.addFile(url)`

`df = spark.read.option("encoding", "UTF-8").csv(SparkFiles.get("amazon_reviews_us_Musical_Instruments_v1_00.tsv.gz"), sep="\t", header=True, inferSchema=True)`

`df.show()`

I created four specific dataframes which matched the Postgres database schema to be created in pgAdmin.







## Results

From our analysis, it was determined that overall (paid and unapid) there was 14537 total reviews.
60 of these reviews were through the Amazon Vine program (paid reviews). 14477 of these reviews were unpaid.
Out of all the Vine reviews, 34 of them had five star ratings. This meant that 57% of the paid reviews had five star ratings.
Out of all the unpaid reviews, 8212 of them had five star ratings.


##