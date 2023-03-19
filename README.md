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
The following analysis was carried out in the `Amazon_Reviews_ETL.ipynb` file.

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

**Write review_id_df to table in RDS**

`review_id_df.write.jdbc(url=jdbc_url, table='review_id_table', mode=mode, properties=config)`

When selecting the table in SQL, `SELECT * FROM review_id_table`, we see the following table.

![sql_review_id](https://github.com/willmino/Amazon_Vine_Analysis/blob/main/images/sql_review_id.png)

**Write products_df to table in RDS**

`products_df.write.jdbc(url=jdbc_url, table='products_table', mode=mode, properties=config)`

When selecting the table in SQL, `SELECT * FROM products_table`, we see the following table.

![sql_products](https://github.com/willmino/Amazon_Vine_Analysis/blob/main/images/sql_products.png)

**Write customers_df to table in RDS**

`customers_df.write.jdbc(url=jdbc_url, table='customers_table', mode=mode, properties=config)`

When selecting the table in SQL, `SELECT * FROM customers_table`, we see the following table.

![sql_customers](https://github.com/willmino/Amazon_Vine_Analysis/blob/main/images/sql_customers.png)

**Write vine_df to table in RDS**

`vine_df.write.jdbc(url=jdbc_url, table='vine_table', mode=mode, properties=config)`

When selecting the table in SQL, `SELECT * FROM vine_table`, we see the following table.

![sql_vine](https://github.com/willmino/Amazon_Vine_Analysis/blob/main/images/sql_vine.png)

From pgAdmin Postgres, the vine-table was exported as the csv file `vine_table.csv`.
The file was then loaded into a new file called `Vine_Review_Analysis.ipynb` for Python Pandas data calculations.

### Pandas Analysis of the vine_table.csv file

Some critical key performance indicators were required for our client's Amazon reviews, such as total number of reviews, total number of paid reviews,
total number of unpaid reviews, percentage of paid reviews that had a five star rating, and the percentage of unpaid reviews that had a five star rating.

First the data was standardized for only Amazon products which had greater than 20 comments and at least a 50% helpful votes ratio (compared to all rating votes).
These parameters were calculated with the following code:

`reviews_df = df.loc[df["total_votes"]>=20]`

`reviews_df.head()`

`helpful_reviews_df = reviews_df.loc[(reviews_df["helpful_votes"]/reviews_df["total_votes"])>=0.5]`

`helpful_reviews_df.head()`

A variable was created to represent only reviews that were paid for through the Amazon Vine review service:

`vine_reviews_df = helpful_reviews_df.loc[helpful_reviews_df["vine"] == "Y"]`

`vine_reviews_df.head()`

A variable for only unpaid reviews was created with this code:

`non_vine_reviews_df = helpful_reviews_df.loc[helpful_reviews_df["vine"] == "N"]`

`non_vine_reviews_df.head()`

A count for the total number of all reviews (paid and unpaid) was performed with this code:

`all_reviews = helpful_reviews_df["review_id"].nunique()`

Finally, the code to get total number of paid reviews, number of paid five star reviews, and the percentage of paid reviews with five stars was calculated with the below code: 

`# Total number of paid reviews`

`total_paid_reviews = vine_reviews_df["review_id"].nunique()`

`# Number of paid 5-star reviews`

`paid_five_star_reviews = vine_reviews_df.loc[vine_reviews_df["star_rating"]==5.0]["star_rating"].count()`

`# The percentage of paid reviews that are five stars`

`percentage_paid_five_star_reviews = round(100*paid_five_star_reviews/total_paid_reviews)`

Finally, the code to get total number of unpaid reviews, number of unpaid five star reviews, and the percentage of unpaid reviews with five stars was calculated with the below code: 

`# Total number of unpaid reviews`

`total_unpaid_reviews = non_vine_reviews_df["review_id"].nunique()`

`# Number of unpaid 5-star reviews`

`unpaid_five_star_reviews = non_vine_reviews_df.loc[non_vine_reviews_df["star_rating"]==5.0]["star_rating"].count()`

`# The percentage of unpaid reviews that are five stars`

`percentage_unpaid_five_star_reviews = round(100*unpaid_five_star_reviews/total_unpaid_reviews)`

## Results for Musical Instruments Dataset

From our analysis, it was determined that overall (paid and unapid) there was 14537 total reviews.

### Three questions answered by the Results

- There were 60 Vine reviews for musical instruments.
  There were 14477 non-Vine reviews (unpaid reviews). 

- 34 Vine reviews had five star ratings.
  8212 non-Vine reviews had five star ratings.

- This meant that 57% of the paid reviews had five star ratings.
  This also equated to a 57% five star rating percentage out of all unpaid reviews.
## Summary

A large dataset of Amazon customer reviews was extracted from an AWS hosting server. The data was pulled into the cloud-based Google Colab software.
Using PySpark, a software similar to Pandas, the data was appropriately manipulated to match the schema of our Postgres database in pgAdmin.
We used PySpark to successfully transfer the data from Google Colab to pgAdmin. The data on the Postgres server in pgAdmin was hosted by my own AWS database.
We finished off the analysis by performing some calculations on the dataset in Python Pandas.

We determined that the paid reviews five star percentage and the unpaid review five star rating percentage were both approximately 57%.
Evidence for customer review bias would be supported by a higher five-star rating for paid customer reviews compared to unpaid customer reviews.
However, we did not see a discrepancy in the five star rating review frequency between paid and unpaid reviews.
Thus, there was no bias towards paid customer reviews for Amazon products through the Vine service.

### Suggested Additional Analysis

I would perform additional analysis on more Amazon customer review datasets with different product types. If it was determined that specific products with paid customer reviews had a higher five-star rating frequency than the unpaid customer review five-star rating frequency, then we would have evidence for bias toward paid customer positive reviews.

For musical instruments specifically, most people giving the reviews, especially customers paid through the Vine service, are likely to have a minimum competency in the playing of a musical instrument. Since we filtered for "useful" customer reviews with greater than a 50% useful rating out of total product ratings, these comments are likely to be genuine and less biased. This is likely the case for specific product types like musical instruments. For other product types, which don't require a high customer skill cap, such as school supplies, its possible that this dataset of reviews will have more unskilled people using them that are likely to have a less critically informed opinions. This could have a greater chance in leading to more unnecessary negative or positive reviews. Its important to test a variety of different types of products to highlight different factors which could apply to certain product types but not all product types. In doing so, we will tend to approach a stronger evidence based prediction as to whether or not paid customer reviews (for all product types) exhibit some kind of positivity bias relative to unpaid reviews.
