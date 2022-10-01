# Amazon Vine Analysis

## Overview 

The purpose of this project was to perform an analysis on Amazon reviews written by members of the paid Amazon Vine program, which is a service that allows manufacturers and publishers to receive reviews for their products. Some companies will pay Amazon a small fee to provide products to Amazon Vine members, in which that member are expected and required to publish a review about that product. I decided to extract a dataset with reviews for various video games, and used that dataset to determine whether the paid vine users illicited any bias towards their star ratings on video game products, versus the unpaid vine users star ratings.  

## Resources 

* AWS Free Tier Services (RDS, S3)
* Anaconda (Jupyter Notebook and Pandas)
* Amazon Colaboratory Notebook
* PySpark 3.3.0
* PostgreSQL 13
* pgAdmin 4
* datasets: 
  * [Video Game Review](https://s3.amazonaws.com/amazon-reviews-pds/tsv/index.txt) = https://s3.amazonaws.com/amazon-reviews-pds/tsv/amazon_reviews_us_Video_Games_v1_00.tsv.gz
  * [vine_table.csv](https://github.com/Lucky777b/Amazon_Vine_Analysis/blob/main/Resources/vine_table.csv)

## Results 

The steps taken to be able to perform this analysis using an ETL process: 

1. Write schema in PostgreSQL and run query to create 4 tables: 'customer_tables', 'products_table', 'review_id_table', 'vine_table'. 

![SQL Tables](https://github.com/Lucky777b/Amazon_Vine_Analysis/blob/main/Resources/SQL_create_tables.png)

2. Extract an Amazon Review dataset into the Amazon Colaboratory notebook using PySpark.

![Extract](https://github.com/Lucky777b/Amazon_Vine_Analysis/blob/main/Resources/extract_dataset_pyspark.png)

3. Transform the data using PySpark functions

   a. To create the 'customer_table_df' dataframe, I used the groupby() function on the 'customer_id' column, and then chained the agg() function to my groupby() function, which created a new column, 'count(customer_id)'. The agg() function was used to count all the customer ids. Then, I had to rename the 'count(customer_id)' column using the withColumnRenamed() function, so that the column name would match the schema created in pgAdmin. 

![customer_table_df](https://github.com/Lucky777b/Amazon_Vine_Analysis/blob/main/Resources/customers_tableDF.png)

   b. I created my 'products_df' dataframe using the select() function, to transform the data by selecting only the 'product_id' and 'product_title' columns. Then, I used a drop_duplicates() function to retrieve only the unique product ids. 

![products_table_df](https://github.com/Lucky777b/Amazon_Vine_Analysis/blob/main/Resources/products_df.png)

   c. I used the select() function again to create my 'review_id_df', but I had to use the to_date() function to convert the date datatype to 'yyyy-MM-dd' format. 

![review_id_df](https://github.com/Lucky777b/Amazon_Vine_Analysis/blob/main/Resources/review_id_df.png)

   d. I created the 'vine_df' dataframe, by again, using the select() function. 

![vine_df](https://github.com/Lucky777b/Amazon_Vine_Analysis/blob/main/Resources/vine_df.png)


4. Connect to an AWS RDS instance

In order to connect to my AWS RDS instance, I had to create a database on AWS Free Tier, and then retrieve an endpoint link, which was used in the 'jdbc_url' connection string to the database. 

5. Load the transformed data into tables in pgAdmin

By using the '.write.jdbc' method, that takes in certain parameters, I was able to take the cleaned dataframes and write them directly to my PostgreSQL database in pgAdmin. 

![tables_RDS](https://github.com/Lucky777b/Amazon_Vine_Analysis/blob/main/Resources/tables_RDS.png)

Then, I verified that the tables were loaded into my pgAdmin database using 'SELECT * FROM <table_name>' query for each of my 4 tables(as shown below): 

![review_id](https://github.com/Lucky777b/Amazon_Vine_Analysis/blob/main/Resources/review_id_table.png)

![products_table](https://github.com/Lucky777b/Amazon_Vine_Analysis/blob/main/Resources/products_table.png)

![customers_table](https://github.com/Lucky777b/Amazon_Vine_Analysis/blob/main/Resources/customers_table.png)

![vine_table](https://github.com/Lucky777b/Amazon_Vine_Analysis/blob/main/Resources/vine_table.png)


6. Use Pandas to determine whether there is any bias towards favorable reviews from Vine members in the dataset. 

Then, I created a new .ipynb file in jupyter notebook, and used Pandas to filter my dataframe that I could then use to form conclusions about a possible bias on star ratings provided by paid vine members and unpaid amazon product reviewers. 

![paid_vs_unpaid](https://github.com/Lucky777b/Amazon_Vine_Analysis/blob/main/Resources/paid_vs_unpaid.png)

* The total number of product reviews for vine and non-vine users = 40565.
* The total number of paid vine reviews = 94, and the total number of unpaid vine reviews = 40471. 

![total_5star](https://github.com/Lucky777b/Amazon_Vine_Analysis/blob/main/Resources/total_5star.png)

* The total number of 5-Star reviews for paid vine and unpaid non-vine users = 15711. 

![paid_vs_unpaid_5star](https://github.com/Lucky777b/Amazon_Vine_Analysis/blob/main/Resources/paid_vs_unpaid_5star.png)

* The total number of paid vine 5-star reviews = 48, and the total number of unpaid vine 5-star reviews = 15663. 

![percentage_5star](https://github.com/Lucky777b/Amazon_Vine_Analysis/blob/main/Resources/percentage_5star_paidUnpaid.png)

* By dividing the paid 5-star review count by all of the paid reviews(star ratings between 0-5), 51.0% of paid vine members provided a 5-star review.
* By dividing the unpaid 5-star review count by all of the unpaid reviews(star ratings between 0-5), 38.7% of unpaid users provided a 5-star review.

## Analysis 

Based on the results of this analysis, it can be concluded that there is a positivity bias for the reviews provided in the Vine program. These results are rather misleading though, because of the fact that only 94 reviews accounted for the paid vine members, while the total number of unpaid vine reviews was equal to 40,471. Even though, the amount of reviews provided from the vine program is much lower than the non-vine members, the percentage of paid vine members who gave a 5-star review was 12.3% higher than those were not paid. Again, this analysis by itself makes it look like there is a positivity bias, but it leads one to wonder if that number would change if the amount of paid vine reviews was much higher. The low amount of paid vine members brings up another point though, that the vine member program is probably not very popular, and due to that reason, it might not be worth it for companies to pay Amazon to provide their products to paid reviewers. 

![additional_analysis](https://github.com/Lucky777b/Amazon_Vine_Analysis/blob/main/Resources/additional_analysis.png)

An additional analysis could be done by comparing the average overall star ratings for both Vine(paid) and Non-Vine(unpaid) reviews. This analysis could provide information about what the average star ratings were for both groups, as opposed to just providing a percentage for how many of those users provided only 5 star ratings. By looking at this type of analysis, it can provide further support that there is a positivity bias for those paid to review a product versus those who are not. The paid reviews provided higher star ratings, (average rating = 4.20) than the unpaid reviewers (average rating = 3.35), as shown in the image above. 

