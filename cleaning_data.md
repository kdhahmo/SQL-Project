# What issues will you address by cleaning the data?
- changing datatype to be able to work with the columns.
- taking care of NULL values
- normalizing cases
- converting certain money related amounts to their proper form
 - they are varchar at base
  - varchar datatype
  - cast to float (to maintain decimals after the next step)
  - divide by 1,000,000 
  - cast to numeric with 2 decimal points (displaying it the way currency normally is)


# Queries:
## Below, provide the SQL queries you used to clean your data.

## all_sessions


-keeping fullvisitorid as varchar to keep the leading zeroes present in some of the IDs


-Change the syntax on the value for '(Other)' in channelgrouping
	-match the syntax of the other groups
``` SQL
UPDATE
	all_sessions
SET
	channelgrouping  = 'Other'
WHERE
	channelgrouping = '(Other)'
``` 
-Change the column name 'time' to something more descriptive.
```SQL
ALTER TABLE 
	all_sessions
RENAME COLUMN 
	time
to 
	sessiontime
```
-Change the value for countries with a '(not set)' value
``` SQL

UPDATE
	all_sessions
SET
	country  = 'Not Provided'
WHERE
	country = '(not set)'
```
-Change the output value for cities that do not have a city value 
 -change the (not set) to 'Not Provided'
 -change the 'not available in demo dataset' to 'Not Available in Demo Dataset'
``` SQL
UPDATE
	all_sessions
SET
	city  = 'Not Provided'
WHERE
	city = '(not set)'

UPDATE
	all_sessions
SET
	city  = 'Not Available in Demo Dataset'
WHERE
	city = 'not available in demo dataset'
```
-convert totaltransactionrevenue
``` SQL
ALTER TABLE
	all_sessions
ALTER COLUMN 
	totaltransactionrevenue 
TYPE
	FLOAT
USING
	totaltransactionrevenue::FLOAT	


UPDATE
	all_sessions
SET
	totaltransactionrevenue  = totaltransactionrevenue /1000000
WHERE
	totaltransactionrevenue  > 0


ALTER TABLE 
	all_sessions
ALTER COLUMN 
	totaltransactionrevenue 
TYPE
	numeric(10,2)
USING
	totaltransactionrevenue::numeric(10,2)
```
-change transactions to integer
``` SQL
ALTER TABLE
	all_sessions
ALTER COLUMN 
	transactions 
TYPE
	INT
USING
	transactions::INT	
```

-change pageviews to integer
``` SQL
ALTER TABLE
	all_sessions
ALTER COLUMN 
	pageviews 
TYPE
	INT
USING
	pageviews::INT
```

-change sessionqualitydim to integer
``` SQL
ALTER TABLE
	all_sessions
ALTER COLUMN 
	sessionqualitydim 
TYPE
	INT
USING
	sessionqualitydim::INT
```
-'varchar' column is called 'date' in file we received
 -changing to 'sessiondate' in order to avoid potential issues with the built in DATE call and to be more descriptive.
``` SQL
ALTER TABLE 
	all_sessions
RENAME COLUMN 
	varchar 
to 
	sessiondate
--also CAST the data type to date

ALTER TABLE
	all_sessions
ALTER COLUMN 
	sessiondate 
TYPE
	DATE
USING
	sessiondate::DATE
```
-change type syntax
``` SQL
UPDATE
	all_sessions
SET
	type = (SUBSTRING (type, 1, 1) || LOWER(SUBSTRING (type, 2)))
```
-change productrefundamount to numeric
``` SQL
ALTER TABLE 
	all_sessions
ALTER COLUMN 
	productrefundamount 
TYPE
	numeric(10,2)
USING
	productrefundamount::numeric(10,2)
```
-change productquantity to integer
``` SQL
ALTER TABLE 
	all_sessions
ALTER COLUMN 
	productquantity 
TYPE
	INT
USING
	productquantity::INT

UPDATE
	all_sessions
SET
	productquantity = 0
WHERE
	productquantity IS NULL
```
-make the appropriate conversions and changes for productprice
``` SQL
ALTER TABLE
	all_sessions
ALTER COLUMN 
	productprice
TYPE
	FLOAT
USING
	productprice::FLOAT	
	
UPDATE
	all_sessions
SET
	productprice = productprice/1000000
WHERE
	productprice > 0
	
ALTER TABLE 
	all_sessions
ALTER COLUMN 
	productprice
TYPE
	numeric(10,2)
USING
	productprice::numeric(10,2)
```

-make the appropriate conversions and changes for productrevenue
``` SQL
ALTER TABLE
	all_sessions
ALTER COLUMN 
	productrevenue
TYPE
	FLOAT
USING
	productrevenue::FLOAT	

UPDATE
	all_sessions
SET
	productrevenue = productrevenue/1000000
WHERE
	productrevenue > 0

ALTER TABLE 
	all_sessions
ALTER COLUMN 
	productrevenue
TYPE
	numeric(10,2)
USING
	productrevenue::numeric(10,2)
```

-change null values in itemquantity, itemrevenue, transactionrevenue, and transactionid
 -alter the data types in order to make it easier to work with the data
``` SQL
ALTER TABLE 
	all_sessions
ALTER COLUMN 
	itemquantity
TYPE 
	INT
USING
	itemquantity::INT

--all null values, so no need to do the longer process here
ALTER TABLE 
	all_sessions
ALTER COLUMN 
	itemrevenue
TYPE 
	numeric(10,2)
USING
	itemrevenue::numeric(10,2)
	
ALTER TABLE 
	all_sessions
ALTER COLUMN 
	transactionrevenue
TYPE
	FLOAT	
USING
	transactionrevenue::FLOAT	



UPDATE
	all_sessions
SET
	transactionrevenue = transactionrevenue/1000000
WHERE
	transactionrevenue > 0



ALTER TABLE 
	all_sessions
ALTER COLUMN 
	transactionrevenue 
TYPE
	numeric(10,2)
USING
	transactionrevenue::numeric(10,2)


	ALTER TABLE 
	all_sessions
ALTER COLUMN 
	ecommerceaction_type
TYPE
	INT
USING
	ecommerceaction_type::INT


ALTER TABLE 
	all_sessions
ALTER COLUMN 
	ecommerceaction_step
TYPE
	INT
USING
	ecommerceaction_step::INT


UPDATE
	all_sessions
SET
	totaltransactionrevenue = 0
WHERE
	totaltransactionrevenue IS NULL

UPDATE
	all_sessions
SET
	transactionrevenue = 0
WHERE
	transactionrevenue IS NULL

```

## analytics


-change visitnumber to integer
```SQL
ALTER TABLE 
	analytics
ALTER COLUMN 
	visitnumber 
TYPE
	INT
USING
	visitnumber::INT
```
-change 'date' column to be more descriptive and avoid potential issues.
```SQL
ALTER TABLE 
	analytics 
RENAME COLUMN 
	date 
TO 
	visitdate;

ALTER TABLE 
	analytics
ALTER COLUMN 
	 visitdate
TYPE
	DATE
USING
	visitdate::DATE
```

-normalize channelgrouping values
```SQL
UPDATE
	analytics
SET
	channelgrouping = 'Other'
WHERE
	channelgrouping = '(Other)'
```
-change data types to better work with the data
```SQL
ALTER TABLE 
	analytics
ALTER COLUMN 
	 units_sold 
TYPE
	INT
USING
	units_sold::INT


ALTER TABLE 
	analytics
ALTER COLUMN 
	 pageviews
TYPE
	INT
USING
	pageviews::INT

--bounces only has 1 for columns with values. a bounce ends a session, so bounces will only ever equal 1 or 0.

Update
	analytics
SET
	bounces = 0
WHERE
	bounces IS NULL

ALTER TABLE 
	analytics
ALTER COLUMN 
	 bounces
TYPE
	BOOLEAN
USING
	bounces::BOOLEAN

--converting more money data 

ALTER TABLE
	analytics
ALTER COLUMN 
	revenue 
TYPE
	FLOAT
USING
	revenue::FLOAT	


UPDATE
	analytics
SET
	revenue = revenue/1000000
WHERE
	revenue > 0

UPDATE
	analytics
SET
	revenue = 0
WHERE
	revenue IS NULL


ALTER TABLE 
	analytics
ALTER COLUMN 
	revenue 
TYPE
	numeric(10,2)
USING
	revenue::numeric(10,2)



ALTER TABLE
	analytics
ALTER COLUMN 
	unit_price 
TYPE
	FLOAT
USING
	unit_price::FLOAT


UPDATE
	analytics
SET
	unit_price = unit_price/1000000
WHERE
	unit_price > 0

ALTER TABLE 
	analytics
ALTER COLUMN 
	unit_price 
TYPE
	numeric(10,2)
USING
	unit_price::numeric(10,2)

```
## products

```SQL
--sku has all distinct values. 
ALTER TABLE products
ADD PRIMARY KEY (sku)
	

ALTER TABLE
	products
ALTER COLUMN 
	orderedquantity  
TYPE
	INT
USING
	orderedquantity::INT


ALTER TABLE
	products
ALTER COLUMN 
	stocklevel   
TYPE
	INT
USING
	stocklevel::INT	

ALTER TABLE
	products
ALTER COLUMN 
	restockingleadtime   
TYPE
	INT
USING
	restockingleadtime::INT	

ALTER TABLE
	products
ALTER COLUMN 
	sentimentscore   
TYPE
	FLOAT
USING
	sentimentscore::FLOAT


ALTER TABLE
	products
ALTER COLUMN 
	sentimentmagnitude   
TYPE
	FLOAT
USING
	sentimentmagnitude::FLOAT	
```
## sales_by_sku
```SQL
ALTER TABLE
	sales_by_sku
ALTER COLUMN 
	total_ordered    
TYPE
	INT
USING
	total_ordered::INT
```

## sales_report
``` SQL
--productsku values are all distinct.
ALTER TABLE sales_report
ADD PRIMARY KEY (productsku)

ALTER TABLE
	sales_report
ALTER COLUMN 
	total_ordered    
TYPE
	INT
USING
	total_ordered::INT

ALTER TABLE
	sales_report
ALTER COLUMN 
	stocklevel     
TYPE
	INT
USING
	stocklevel::INT

ALTER TABLE
	sales_report
ALTER COLUMN 
	restockingleadtime     
TYPE
	INT
USING
	restockingleadtime::INT

ALTER TABLE
	sales_report
ALTER COLUMN 
	sentimentscore     
TYPE
	FLOAT
USING
	sentimentscore::FLOAT

ALTER TABLE
	sales_report
ALTER COLUMN 
	sentimentmagnitude     
TYPE
	FLOAT
USING
	sentimentmagnitude::FLOAT


ALTER TABLE
	sales_report
ALTER COLUMN 
	ratio     
TYPE
	FLOAT
USING
	ratio::FLOAT

```

## visitors
``` SQL
CREATE TABLE visitors(--using data downloaded from the Google Analytics Sample dataset to tie together hits and the timing of visits to the site with the rest of the data.
visitNumber VARCHAR
, visitId VARCHAR
, fullVisitorId VARCHAR
, date VARCHAR
, hits VARCHAR
, hitNumber VARCHAR
, time VARCHAR
, hour VARCHAR
, minute VARCHAR
);

TRUNCATE TABLE visitors
RESTART IDENTITY;
--preparing the table for importing a csv with columns matching the table into the database. 
--formatting the data
ALTER TABLE
	visitors
ALTER COLUMN 
	visitnumber     
TYPE
	INT
USING
	visitnumber::INT


ALTER TABLE 
	visitors
RENAME COLUMN 
	date 
to 
	visitdate


ALTER TABLE
	visitors
ALTER COLUMN 
	visitdate     
TYPE
	DATE
USING
	visitdate::DATE

ALTER TABLE
	visitors
ALTER COLUMN 
	hits     
TYPE
	INT
USING
	hits::INT


ALTER TABLE
	visitors
ALTER COLUMN 
	hitnumber     
TYPE
	INT
USING
	hitnumber::INT

ALTER TABLE 
	visitors
RENAME COLUMN 
	time--renaming for more description and to avoid potential errors from having same name as a preset
to 
	visittime

--casting visittime to integer to make it easier to work with

ALTER TABLE
	visitors
ALTER COLUMN 
	visittime     
TYPE
	INT
USING
	visittime::INT


ALTER TABLE 
	visitors
RENAME COLUMN 
	hour--renaming for more description and to avoid potential errors from having same name as a preset

to 
	visithour


ALTER TABLE
	visitors
ALTER COLUMN 
	visithour     
TYPE
	INT
USING
	visithour::INT



ALTER TABLE 
	visitors
RENAME COLUMN 
	minute--renaming for more description and to avoid potential errors from having same name as a preset

to 
	visitminute


ALTER TABLE
	visitors
ALTER COLUMN 
	visitminute     
TYPE
	INT
USING
	visitminute::INT

```
## visit_hits
``` SQL
CREATE TABLE visit_hits--creating a table with hit data to work with
AS
(SELECT
	DISTINCT fullvisitorid--want DISTINCT so this table can have a composite primary key.
	, visitid
	, SUM(hits) OVER(--this is based on distinct combos of fullvisitorid and visitid, not the other way around.
	PARTITION BY fullvisitorid, visitid
	ORDER BY fullvisitorid
	) AS totalhits
FROM(
	SELECT
		DISTINCT fullvisitorid
		, visitid
		, hits
	FROM
		visitors
	)
)

ALTER TABLE visit_hits
ADD PRIMARY KEY (fullvisitorid, visitid)
```
