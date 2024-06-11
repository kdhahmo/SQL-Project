# Final-Project-Transforming-and-Analyzing-Data-with-SQL

## Project/Goals

We were provided with 5 csv files and given instructions on how to work with the data in them.
The data is an obfuscated Google Analytics Sample from the Google Merchandise Store from August 2016 - August 2017.
Goals:
- load the csv files into the database
- clean the data
- answer the provided questions about the data
- come up with more questions that the data can answer and answer them
- QA the data
- generate the ERD of the database

[assignment.md has an overview of the process](/assignment.md)

## Process
### Load the Data
- Download the provided files
- Go to pgAdmin4
 - Go to the Object Explorer sidebar
 - Right click on 'Databases' and click 'Create >' 'Database...'
 - Name the database 'ecommerce'
 - Set up tables by using the following template. Because many of the tables don't have a column or a combination of columns with all unique values, we will be omitting a Primary Key during the setup of the tables. When we clean the data we can verify if a column or combination of two columns would work for any given table by checking for duplicates.
```SQL
CREATE TABLE persons (
  column_name1 VARCHAR,--match the name to what is shown in the csv file. use varchar for all columns in order to smoothly transfer the data.
  column_name2 VARCHAR--use as many columns as you need. 
);


TRUNCATE TABLE persons 
RESTART IDENTITY;
```
 - In the Object Explorer sidebar, right click a table and select 'Import/Export...'
   - Make sure the toggle is on 'Import'
   - Make sure the 'Header' option is toggled on.
   - Choose the csv file for that table.
   - Ensure the columns match those you have set for the table.
   - Import the csv file.
 - Repeat this process for every table. 

### Examine the Data and Determine the Data Source
The data was provided with no comment as to its source, so the next step is to see what information we can get from the files at the [source we have been provided with](https://drive.google.com/drive/folders/1efDA4oc9w-bTbAvrESdOJpg9u-gEUBhJ). There are no descriptions on the files, so the next step is to check online.

The search 'data set fullVisitorId, channelgrouping, time, country, city' turned up posts mentioning an 'Ecommerce' or 'E-Commerce' dataset and 'BigQuery'. Searching '"E-Commerce" retail data set "google analytics”' leads to a github page that draws from a google analytics data source. The github page links to a [BigQuery schema](https://support.google.com/analytics/answer/3437719?hl=en).

Above the '[UA] BigQuery Export schema' title are links. The [BigQuery Export](https://support.google.com/analytics/topic/3416089?hl=en&ref_topic=2430414&sjid=7886829371119872236-NC) link has a page titled [[UA] Google Analytics sample dataset for BigQuery](https://support.google.com/analytics/answer/7586738?hl=en&ref_topic=3416089&sjid=7886829371119872236-NC#zippy=%2Cin-this-article). This page provides information on the dataset that we are working with. We can verify it is the same dataset by checking the column values and date range. Both the provided data and this dataset have dates only in the range of August 2016 to August 2017. The types of data in this sample database match up with what we can see from the provided data. Column names like fullvisitorid, visitid, and ecommerceaction_type are the same.

This page explains that the data is obfuscated Google Analytics 360 data from the Google Merchandise Store. It explains that some data such as fullvisitorids have been obfuscated and others have been removed and either made null in the case of integers or “Not available in demo dataset” in the case of string values. This helps us interpret “Not available in demo dataset” values in the city column.

[See the Come Up With More Questions About the Data and Answer Them section in Results for details on the data.](https://github.com/kdhahmo/SQL-Project?tab=readme-ov-file#come-up-with-more-questions-about-the-data-and-answer-them)

### Clean the Data
Because all columns were imported as the varchar data type, many of the columns need to have their datatypes cast to the correct ones. 
Columns with money values need to have their numbers divided by 1,000,000. 
Other things like syntax need to be addressed. 

[See the Clean the data section in Results for an overview of the cleaning process.](https://github.com/kdhahmo/SQL-Project?tab=readme-ov-file#clean-the-data)

[Steps are provided in cleaning_data.md](/cleaning_data.md)

### Answer the Provided Questions About the Data
The data can tell us about the cities and countries of people who visited the site.

[See the Starting With Questions section in Results for an overview of these questions and answers.](https://github.com/kdhahmo/SQL-Project?tab=readme-ov-file#starting-with-questions)

[See starting_with_questions.md for in depth process on the questions and answers](/starting_with_questions.md)

### Come Up With More Questions About the Data and Answer Them
The data can tell us about the total hits across the site.
We will query the online dataset to tie more data together.
```SQL
SELECT
	visitNumber
	, visitId--identifying information that we can use to tie this to the data we were provided
	, fullVisitorId--identifying information that we can use to tie this to the data we were provided
	, date
	, totals.hits
	, hits.hitNumber
	, hits.time
	, hits.hour
	, hits.minute
FROM
	`bigquery-public-data.google_analytics_sample.ga_sessions_*`
	, UNNEST(hits) AS hits
	
WHERE
  _TABLE_SUFFIX BETWEEN '20160801' AND '20170801'--in BigQuery the data is separated by year and month. this lets us put all of our results in one table.

```
[See the Starting With Data section in Results for an overview of these questions and answers.](https://github.com/kdhahmo/SQL-Project?tab=readme-ov-file#starting-with-data)

[See starting_with_data.md for in depth process on the questions and answers](/starting_with_data.md)

### QA the Data
This dataset has been purposefuly obfuscated to protect privacy. While there are some values that are explicitly said to have been changed, there are still uncertainties with using this data.

[See the Challenges and QA section in Results for an overview of the QA process.](https://github.com/kdhahmo/SQL-Project?tab=readme-ov-file#challenges-and-qa)

[See QA.md for in depth QA process](/QA.md)

### Generate the ERD of the Database
The ERD can be generated by going to the Object Explorer sidebar, right clicking on the 'ecommerce' database, and clicking on 'ERD for database'.

[The ERD is saved as schema.png](/schema.png)


## Results
### Starting With Questions
The data can tell us about the cities and countries of people who visited the site. 


We can find the highest transaction revenues by city and country by using the city and country information stored in the all_sessions table and the revenue information stored in the analytics table. We can join the two tables using the fullvisitorid column and group the data by city or country and get the sum of the revenue.

We can find the average number of products ordered from visitors in each city and country on the site by using the city, country, transactionrevenue, and productquantity information in the all_sessions table. We can group the data by city or country and get the average number of products ordered.

We can determine if there is any pattern in the types (product categories) of products ordered from visitors in each city and country by using the city, country, transactionrevenue, and productquantity information in the all_sessions table. We can group the data by city or country and get the average number of products ordered and use transactionrevenue as a filter to only show pages where the product looked at was ordered.

We can determine the top-selling product from each city/country and determine if there is a pattern by using the v2productname, city, country, transactionrevenue and productsku information in the all_sessions table. We can group the data by city or country, v2productname, and productsku and get the sum of transactionavenue by these combined categories. We can use this value to determine the top seller and filter out only the highest selling product for each city or country.

We can summarize the impact of revenue generated from each city/country by using the city and country information in the all_sessions table and the revenue information in the analytics table. We can join the two tables using the fullvisitorid column and group the data by city or country and get the sum of the revenue. 

[See starting_with_questions.md for in depth process on the questions and answers](/starting_with_questions.md)


### Starting With Data
The data can tell us about the total hits across the site.

Querying extra data from the data source to answer these questions.
``` SQL
SELECT
	visitNumber
	, visitId
	, fullVisitorId
	, date
	, totals.hits
	, hits.hitNumber
	, hits.time
	, hits.hour
	, hits.minute
FROM
	`bigquery-public-data.google_analytics_sample.ga_sessions_*`
	, UNNEST(hits) AS hits
	
WHERE
  _TABLE_SUFFIX BETWEEN '20160801' AND '20170801'
```

We can find how many unique visitors were there by month by using the sessiondate and fullvisitorid in the all_sessions table. We can extract the year and month from sessiondate into separate columns to group the data by. We can then count distinct fullvisitorids from this grouped data. 

We can find what cities and countries have the highest amount of hits by using the city and country information in the all_sessions table and the totalhits information in the visit_hits table. We can join the two tables using the fullvisitorid column and group the data by city or country. We can then get the sum of totalhits by this grouped data. 

We can find the average amount of hits across sessions that viewed certain product categories by using the v2productcategory information in the all_sessions table and the totalhits information in the visit_hits table. We can join the two tables using the visitid column and group the data by v2productcategory. We can then get the average of totalhits by this grouped data. 

We can determine if there is any pattern between the total hits and total unique visitors for each city/country by using the city, country, and fullvisitorid information in the all_sessions table and the totalhits information in the visit_hits table. We can join the two tables using the visitid column and group the data by city or country. We can then get the sum of totalhits by this grouped data. 

We can determine if there is any pattern between the total hits on sessions where products are viewed and how many of those products are ordered by using the sku and name information in the products table, the totalhits information in the visit_hits table, and the total_ordered information in the sales_by_sku table. We can join the visit_hits table to the all_sessions table by using the visitid column. We can join the all_sessions table to the products table by using the productsku column in the all_sessions table and the sku column in the products table. We can join the products table and the sales_by_sku table using the sku column in the products table and the productsku column in the sales_by_sku table. We can group the data by the product. We can then get the sum of totalhits and total_ordered by this grouped data.

[See starting_with_data.md for in depth process on the questions and answers](/starting_with_data.md)


## Challenges and QA 
The data has a lot of obfuscated values, which makes it hard to interpret it to perform cleaning and analysis on.
These issues with the data make it difficult to link tables together with the use of Primary Keys and Foreign Keys.


If a city or country name is written with different case types, more 'unique' cities could exist than there actually are.

If a revenue value is null, negative, or an extremely large value it could skew data.

If fullvisitorid values have different numbers of characters it could indicate unreliable information. It could also be possible that some data is from different fullvisitorids but the obfuscation has erased this fact.

Many cities aren't labeled. This can lead to skewed data. The actual cities of these data points aren't properly attributed, and the new 'city' values created from these placeholder values makes potentially unrelated data related.

If a productquantity value is null, negative, or an extremely large value it could skew data.

If a totaltransaction value is null, negative, or an extremely large value it could skew data.

If a sessiondate value falls outside of the dates of the Google Analytics Sample dataset that appears to be the data source, there is likely either an issue with inputs or a misunderstanding on the data source.

If a totalhits value is null, negative, or an extremely large value it could skew data.

If a total_ordered value is null, negative, or an extremely large value it could skew data.

[See QA.md for in depth QA process](/QA.md)

## Future Goals
If I had more time, I would look into foreign key constraints and see if I could clean the data in ways to be able to make the productsku column in the all_sessions table a foreign key.

I also want to convert the 'sessiontime' values after I familiarize myself with the data to understand what it represents.

I would also cross-reference pagetitle, pagepathlevel1, ecommerceaction_type, ecommerceaction_step, and ecommerceaction_option to see if there is any missing data that could be filled in, or other labels in any column that would better classify information. I had noticed that some transactions registered as being on earlier ecommerce related steps. If I had the time I would compare the transaction data we do have to see how the ecommerce information relates to others in hopes to try to interpret what the numbers for ecommerceaction_type mean.
