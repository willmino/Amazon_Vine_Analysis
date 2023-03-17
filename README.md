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

I created four new dataframes according to the Postgres database schema to be created in pgAdmin.
Postgres database schema:

![Postgres](https://github.com/willmino/Amazon_Vine_Analysis/blob/main/images/schema.png)

The customers_table dataframe was created with this code: 

`from pyspark.sql.functions import sum,avg,max,count`

`customers_df = df.groupby("customer_id").agg(count("customer_id")).withColumnRenamed("count(customer_id)", "customer_count")`

![Spark_customers](https://github.com/willmino/Amazon_Vine_Analysis/blob/main/images/Spark_customers.png)

The products_table dataframe was created with this code: 

`products_df = df.select(["product_id","product_title"]).drop_duplicates()`

![Spark_products_table](https://github.com/willmino/Amazon_Vine_Analysis/blob/main/images/Spark_products.png)

The review_id dataframe was created with this code:

`review_id_df = df.select(["review_id", "customer_id", "product_id", "product_parent", to_date("review_date", 'yyyy-MM-dd').alias("review_date")])`

![Spark_review_id_table](https://github.com/willmino/Amazon_Vine_Analysis/blob/main/images/Spark_review_id.png)


The vine_table dataframe was created with this code:

`vine_df = df.select(["review_id", "star_rating", "helpful_votes", "total_votes", "vine", "verified_purchase"])`

![Spark_vine_table](https://github.com/willmino/Amazon_Vine_Analysis/blob/main/images/Spark_vine.png)

An AWS RDS instance was created to connect to the Postgres database:

`mode = "append"`

`jdbc_url="jdbc:postgresql://dataviz.cmuktimxt422.us-east-2.rds.amazonaws.com:5432/postgres"`

`config = {"user":"postgres",`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`"password": "#Removed for security reasons",`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`"driver":"org.postgresql.Driver"}`

The four dataframes were then written to the Postgres database through AWS.
# Write review_id_df to table in RDS
`review_id_df.write.jdbc(url=jdbc_url, table='review_id_table', mode=mode, properties=config)`

When selecting the table in SQL, `SELECT * FROM review_id_table`, we see the following table.

![sql_review_id](https://github.com/willmino/Amazon_Vine_Analysis/blob/main/images/sql_review_id.png)

# Write products_df to table in RDS
`products_df.write.jdbc(url=jdbc_url, table='products_table', mode=mode, properties=config)`

When selecting the table in SQL, `SELECT * FROM products_table`, we see the following table.

![sql_products](https://github.com/willmino/Amazon_Vine_Analysis/blob/main/images/sql_products.png)

# Write customers_df to table in RDS
`customers_df.write.jdbc(url=jdbc_url, table='customers_table', mode=mode, properties=config)`

When selecting the table in SQL, `SELECT * FROM customers_table`, we see the following table.

![sql_customers](https://github.com/willmino/Amazon_Vine_Analysis/blob/main/images/sql_customers.png)

# Write vine_df to table in RDS
`vine_df.write.jdbc(url=jdbc_url, table='vine_table', mode=mode, properties=config)`

When selecting the table in SQL, `SELECT * FROM vine_table`, we see the following table.

![sql_vine](https://github.com/willmino/Amazon_Vine_Analysis/blob/main/images/sql_vine.png)


## Results

From our analysis, it was determined that overall (paid and unapid) there was 14537 total reviews.
60 of these reviews were through the Amazon Vine program (paid reviews). 14477 of these reviews were unpaid.
Out of all the Vine reviews, 34 of them had five star ratings. This meant that 57% of the paid reviews had five star ratings.
Out of all the unpaid reviews, 8212 of them had five star ratings.


## Summary