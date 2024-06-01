What are your risk areas? Identify and describe them.
This dataset has been purposefuly obfuscated to protect privacy. While there are some values that are explicitly said to have been changed, there are still uncertainties with using this data.
1. If a city or country name is written with different case types, more 'unique' cities could exist than there actually are.
2. If a revenue value is null, negative, or an extremely large value it could skew data.
3. If fullvisitorid values have different numbers of characters it could indicate unreliable information. It could also be possible that some data is from different fullvisitorids but the obfuscation has erased this fact.
4. Many cities aren't labeled. This can lead to skewed data. The actual cities of these data points aren't properly attributed, and the new 'city' values created from these placeholder values makes potentially unrelated data related.
5. If a productquantity value is null, negative, or an extremely large value it could skew data.
6. If a totaltransaction value is null, negative, or an extremely large value it could skew data.
7. If a sessiondate value falls outside of the dates of the Google Analytics Sample dataset that appears to be the data source, there is likely either an issue with inputs or a misunderstanding on the data source.
8. If a totalhits value is null, negative, or an extremely large value it could skew data.
9. If a total_ordered value is null, negative, or an extremely large value it could skew data.

QA Process:
Describe your QA process and include the SQL queries used to execute it.

1. Ensuring that the city and country values have the same case, to prevent unique values that are actually duplicates.

``` SQL
SELECT
	CASE
	--If DISTINCT number of city values changes when case is normalized, there were different capitalizations (eg. Calgary, calgary, CALGARY)
		WHEN COUNT(DISTINCT(city)) = COUNT(DISTINCT(LOWER(city)))
		THEN 'pass'
		ELSE 'fail'
	END AS citycase
	, CASE
	--If DISTINCT number of country values changes when case is normalized, there were different capitalizations (eg. Canada, canada, CANADA)
		WHEN COUNT(DISTINCT(country)) = COUNT(DISTINCT(LOWER(country)))
		THEN 'pass'
		ELSE 'fail'
	END AS countrycase
FROM
	all_sessions
```
citycase	countrycase
pass	pass
The casing for city and country values isn't creating false distinct values.

2. Checking if the range of revenue for products makes logical sense with no nulls, negatives, or extremely large values.
``` SQL
SELECT
	revenue
FROM--Getting data that has been checked to put a WHERE filter on it.
(
	SELECT
		CASE
	--It doesn't make sense if the revenue is negative or extremely large.
			WHEN revenue IS NULL
			THEN 'fail'
			WHEN revenue < 0 
			THEN 'fail'
			WHEN revenue > 1000000
			THEN 'fail'
			ELSE 'pass'
		END AS revenueqa
		, revenue
	FROM
		analytics
)
--Any values that get returned failed.
WHERE
	revenueqa = 'fail'
```
This assertation returns no values, meaning all revenue data has a value that isn't negative or extremely large. 

3. Checking the range of characters for fullvisitorid. Different numbers of characters could indicate unreliable information.
``` SQL
SELECT
	CASE
	--If the amount of characters used in a visitorid differ, this test will fail.
		WHEN COUNT(DISTINCT(LENGTH(fullvisitorid))) > 1
		THEN 'fail'
		ELSE 'pass'
	END AS fullvisitoridqa
FROM
	all_sessions
```
The data returns as a fail, meaning that the fullvisitorid column has values of varying character lengths. There isn't anything that can be done to verify uniqueness of fullvisitorids in this data, due to the nature of the obfuscation. We can only hope that any given fullvisitorid is unique and that the difference in character length is simply a further obfuscation.


4. Many cities aren't labeled. This can lead to skewed data. The actual cities of these data points aren't properly attributed, and the new 'city' values created from these placeholder values makes potentially unrelated data related.
``` SQL
SELECT--Showing all values on failing rows to show what data is affected.
	*
FROM--Getting data that has been checked to put a WHERE filter on it.
(
	SELECT
		*
		, CASE
		--If a city is given a placeholder it's not properly labeled and could skew the data.
			WHEN city IN ('Not Available in Demo Dataset', 'Not Provided')--'Not Available in Demo Dataset' is the way obfuscated city data was noted. 'Not Provided' is cleaned from '(not set)' which seems to indicate that the data wasn't provided in the first place to be obfuscated.
			THEN 'fail'
			ELSE 'pass'
		END AS citynameqa
	FROM
		all_sessions
)
--Any values that get returned failed.
WHERE
	citynameqa = 'fail'
```
This returns 8656 rows that don't have actual cities recorded. These rows still have useful data in the other columns. If performing queries with city information, these rows can be cleaned from the results using filtering. Such as:
```SQL
WHERE
	city NOT IN ('Not Available in Demo Dataset', 'Not Provided')
```
5. Checking that the productquantity values are in logical ranges.
``` SQL
SELECT--Showing all values on failing rows to show what data is affected.
	*
FROM--Getting data that has been checked to put a WHERE filter on it.
(
	SELECT
	*
	, CASE
	--If a product quantity value is null, negative, or extremely large it will fail.
		WHEN productquantity IS NULL
		THEN 'fail'
		WHEN productquantity < 0
		THEN 'fail'
		WHEN productquantity > 1000000
		THEN 'fail'
		ELSE 'pass'
	END AS productquantityqa
	FROM
		all_sessions
)
--Any values that get returned failed.
WHERE
	productquantityqa = 'fail'
```
No values are returned, so all of the productquantity values are in logical ranges.

6. Checking that the totaltransaction values are in logical ranges.
``` SQL
SELECT--Showing all values on failing rows to show what data is affected.
	*
FROM--Getting data that has been checked to put a WHERE filter on it.
(
	SELECT
	*
	, CASE
	--If a total transaction value is null, negative, or extremely large it will fail.
		WHEN totaltransactionrevenue IS NULL
		THEN 'fail'
		WHEN totaltransactionrevenue < 0
		THEN 'fail'
		WHEN totaltransactionrevenue > 1000000
		THEN 'fail'
		ELSE 'pass'
	END AS totaltransactionrevenueqa
	FROM
		all_sessions
)
--Any values that get returned failed.
WHERE
	totaltransactionrevenueqa = 'fail'
```
No values are returned, so all of the totaltransactionrevenue values are in logical ranges.

7. Checking that sessiondate falls within the dates of the dataset. The Google Analytics Sample that the data comes from has data from August 2016 to August 2017.
``` SQL
SELECT--Showing all values on failing rows to show what data is affected.
	*
FROM--Getting data that has been checked to put a WHERE filter on it.
(
	SELECT
	*
	, CASE
	--The dataset is from August 2016 to August 2017. Any values outside of that will fail.
		WHEN sessiondate IS NULL
		THEN 'fail'
		WHEN sessiondate < '2016-08-01'
		THEN 'fail'
		WHEN sessiondate > '2017-08-01'
		THEN 'fail'
		ELSE 'pass'
	END AS sessiondateqa
	FROM
		all_sessions
)
--Any values that get returned failed.
WHERE
	sessiondateqa = 'fail'
```
No values are returned, so all of the sessiondate values are within those of the dataset this data appears to come from.

8. Checking if totalhits values are logical.
``` SQL
SELECT--Showing all values on failing rows to show what data is affected.
	*
FROM--Getting data that has been checked to put a WHERE filter on it.
(
	SELECT
	*
	, CASE
	--If a total hit value is null, negative, or extremely large it will fail.
		WHEN totalhits IS NULL
		THEN 'fail'
		WHEN totalhits < 0
		THEN 'fail'
		WHEN totalhits > 1000000
		THEN 'fail'
		ELSE 'pass'
	END AS totalhitsqa
	FROM
		visit_hits
)
--Any values that get returned failed.
WHERE
	totalhitsqa = 'fail'
```
No values are returned, so all of the totalhits values are in logical ranges.

9. Checking if total_ordered values are logical.
``` SQL
SELECT--Showing all values on failing rows to show what data is affected.
	*
FROM--Getting data that has been checked to put a WHERE filter on it.
(
	SELECT
	*
	, CASE
	--If a total ordered value is null, negative, or extremely large it will fail.
		WHEN total_ordered IS NULL
		THEN 'fail'
		WHEN total_ordered < 0
		THEN 'fail'
		WHEN total_ordered > 1000000
		THEN 'fail'
		ELSE 'pass'
	END AS total_orderedqa
	FROM
		sales_by_sku
)
--Any values that get returned failed.
WHERE
	total_orderedqa = 'fail'
```
No values are returned, so all of the total_ordered values are in logical ranges.