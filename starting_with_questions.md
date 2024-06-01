cAnswer the following questions and provide the SQL queries used to find the answer.

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**


SQL Queries:
```SQL
--for city
SELECT
	*
FROM 
	(
	SELECT
		DISTINCT alls.city
		, alls.country
		, SUM(an.revenue) OVER (--grouping revenue by city. including country in case of same city name in different country.
			PARTITION BY alls.city, alls.country
		) AS cityrevenue
	FROM
		all_sessions alls
	JOIN
	analytics an
	USING
		(fullvisitorid)
	ORDER BY 
		cityrevenue DESC
	)
WHERE
	cityrevenue > 0
AND 
	city NOT IN ('Not Set', 'Not Available in Demo Dataset')
	--these values get put together but we aren't able to determine what city they actually are, due to how the data was obfuscated. better to leave them out of city focused data.
	
--for country
SELECT
	*
FROM 
	(
	SELECT
		DISTINCT alls.country
		, SUM(an.revenue) OVER (
			PARTITION BY alls.country
		) AS countryrevenue
	FROM
		all_sessions alls
	JOIN
	analytics an
	USING
		(fullvisitorid)
	ORDER BY 
		countryrevenue DESC
	)
WHERE
	countryrevenue > 0
--the city values the weren't properly labeled don't affect country amounts. 
```
Answer:

Removing the placeholder values, the 25 top cities, from most to least revenue, are:


1	Mountain View	United States	10549.51
2	San Bruno	United States	4230.99
3	New York	United States	3485.77
4	Sunnyvale	United States	3039.18
5	Chicago	United States	1383.63
6	Charlotte	United States	1161.32
7	Kirkland	United States	1103.52
8	San Francisco	United States	987.46
9	Los Angeles	United States	942.99
10	Jersey City	United States	933.85
11	Austin	United States	861.57
12	Seattle	United States	796.91
13	Palo Alto	United States	709
14	Milpitas	United States	494.3
15	Toronto	Canada	487.67
16	Ann Arbor	United States	442.21
17	Salem	United States	346.02
18	San Jose	United States	239.74
19	Cambridge	United States	165.42
20	Fremont	United States	124
21	Atlanta	United States	83
22	South San Francisco	United States	82.42
23	Denver	United States	41.98
24	Yokohama	Japan	30.88
25	Zurich	Switzerland	16.99



These cities are the only ones with revenue.

There are 5 countries with revenue. They are:
1	United States	115229.27
2	Canada	487.67
3	Germany	69.98
4	Japan	30.88
5	Switzerland	16.99



**Question 2: What is the average number of products ordered from visitors in each city and country?**

SQL Queries:
```SQL
-- city
--orders were actually placed
SELECT
	DISTINCT city
	, country
	, AVG(productquantity)::INT AS avgproductsordered
FROM
	all_sessions
WHERE
	transactionrevenue > 0 --getting actual transactions
GROUP BY
	city, country


--include everything
SELECT
	DISTINCT city
	, country
	, AVG(productquantity)::INT AS avgproductsordered
FROM
	all_sessions
GROUP BY
	city, country
ORDER BY
	avgproductsordered DESC

--country
SELECT
	DISTINCT country
	, AVG(productquantity)::INT AS avgproductsordered
FROM
	all_sessions
WHERE
	transactionrevenue > 0 --getting actual transactions
GROUP BY
	country

--include everything
SELECT
	DISTINCT country
	, AVG(productquantity)::INT AS avgproductsordered
FROM
	all_sessions
GROUP BY
	country
ORDER BY
	avgproductsordered DESC
```
Answer:
There is 1 city recorded in the data with a transaction between August 2016 and August 2017. The other cities were obfuscated in the data.
The average number of products ordered by visitors from Sunnyvale United States is 1. 
The average number of products ordered by visitors from unnamed cities is 17. 

Accounting for all visitors, only 2 cities had an average order amount over 0. Columbus United States and Madrid Spain both have an average of 1 product ordered when accounting for all visits.

The United States is the only country with transactions between August 2016 and August 2017.
The average number of products ordered by visitors from the United States is 13. 

Accounting for all visitors, no countries had an average order amount over 0.

**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**

SQL Queries:
```SQL
--city
--ordered only
SELECT
	DISTINCT city
	, country
	, v2productcategory
FROM
	all_sessions
WHERE
	transactionrevenue > 0 --getting actual transactions
ORDER BY
	city
	, country
--all sessions
SELECT
	DISTINCT city
	, country
	, v2productcategory
FROM
	all_sessions
ORDER BY
	city
	, country


--country
--all sessions
SELECT
	DISTINCT country
	, v2productcategory
FROM
	all_sessions
ORDER BY
	country
--ordered only
SELECT
	DISTINCT country
	, v2productcategory
FROM
	all_sessions
WHERE
	transactionrevenue > 0 --getting actual transactions
ORDER BY
	country
```
Answer:

There were only a few products actually purchased. Of those products, there were
Not Available in Demo Dataset	United States	Bags
Not Available in Demo Dataset	United States	Electronics
Sunnyvale	United States	Nest-USA


United States	Bags
United States	Electronics
United States	Nest-USA


From all products looked at, each city had a wide variety of products.
Ahmedabad India had products ranging from Google and Youtube brand products to accessories, to bags, to men's apparel, to office supplies.

Countries were similar to cities in their variety. 
Albania had Google and YouTube brand related items and men's apparel. 
Argentina had electronics, brand products, men's apparel, women's apparel, office supplies, backpacks, water bottles, lifestyle products, and accessories.


**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**

SQL Queries:
```SQL
--by city
SELECT
	v2productname
	, city
	, country
	, transactionrevenue
FROM --using a subquery so I can clean up the data I present, and to filter it
(
	SELECT
		v2productname
		, city
		, country
		, SUM(transactionrevenue) AS transactionrevenue
		, productsku --in case multiple different products have the same name. 
		, RANK()--to get top seller.
			OVER (
			PARTITION BY city, country--what it's grouped by.
			ORDER BY SUM(transactionrevenue) DESC) AS rank  --DESC starts with highest value. want to order by what we get for transactionrevenue as the highest value (with all sorted in same group added up)
	FROM
		all_sessions
	WHERE
		transactionrevenue > 0 --getting actual transactions
	GROUP BY
		city
		, country
		, v2productname
		, productsku
)
WHERE
	rank = 1

--by country
SELECT
	v2productname
	, country
	, transactionrevenue
FROM 
(
	SELECT
		v2productname
		, country
		, SUM(transactionrevenue) AS transactionrevenue
		, productsku
		, RANK()
			OVER (
			PARTITION BY country
			ORDER BY SUM(transactionrevenue) DESC) AS rank
	FROM
		all_sessions
	WHERE
		transactionrevenue > 0
	GROUP BY
		country
		, v2productname
		, productsku
)
WHERE
	rank = 1
```
Answer:
The top selling product for Sunnyvale United States is the 'Nest® Cam Indoor Security Camera - USA' with $200.00 sold. The other data points had their city and other geolocational data like latitude and longitude removed in the dataset, so there is no way to make an educated guess as to their location in relation to one another. They could all be from the same city. 
For these cities, the top selling product is the 'Reusable Shopping Bag' with $1015.48 sold.

The United states is the only country with transactions in this dataset.
The top seller for this country is the 'Reusable Shopping Bag' with $1015.48 sold.

**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:
```SQL
--by city
SELECT
	city
	, country
	, totalrevenue
FROM(
	SELECT
		city
		, country
		, totalrevenue
		, ROW_NUMBER () 
			OVER(
			PARTITION BY city, country
			ORDER BY totalrevenue
		) AS ordered
	FROM(
		SELECT
			alls.city
			, alls.country
			, an.revenue
			, SUM(an.revenue) 
				OVER(
				PARTITION BY alls.city, alls.country
				ORDER BY SUM(an.revenue) DESC ) AS totalrevenue

		FROM
			all_sessions alls
		JOIN
			analytics an
		USING
			(fullvisitorid)
		GROUP BY
			alls.city
			, alls.country
			, an.revenue
	)
	)
WHERE 
	ordered = 1
AND
	totalrevenue > 0
AND
	city NOT IN ('Not Available in Demo Dataset')
	--city IN ('Not Available in Demo Dataset') 
	--to get the values where the city data was unavailable in the dataset.

ORDER BY totalrevenue DESC


-country
SELECT
	country
	, totalrevenue
FROM(
	SELECT
		country
		, totalrevenue
		, ROW_NUMBER () 
			OVER(
			PARTITION BY country
			ORDER BY totalrevenue DESC
		) AS ordered
	FROM(
		SELECT
			alls.country
			, an.revenue
			, SUM(an.revenue) 
				OVER(
				PARTITION BY alls.country
				ORDER BY SUM(an.revenue) DESC ) AS totalrevenue

		FROM
			all_sessions alls
		JOIN
			analytics an
		USING
			(fullvisitorid)
		GROUP BY
			alls.country
			, an.revenue
	)
	)
WHERE 
	ordered = 1
AND
	totalrevenue > 0
ORDER BY totalrevenue DESC
```
Answer:
Many of the recorded locations do not have any revenue. Of the ones that do we can see the following patterns


There was data unavailable in the dataset that creates a city group that may or may not be the same location.
Removing these data points 

1	Not Available in Demo Dataset	United States	2499
2	Not Available in Demo Dataset	Germany	56.49


we are left with. 

1	San Bruno	United States	1153.5
2	Mountain View	United States	1057.8
3	Kirkland	United States	476.5
4	Milpitas	United States	400
5	Jersey City	United States	379.85
6	New York	United States	376.4
7	San Francisco	United States	359
8	Sunnyvale	United States	358
9	Chicago	United States	312.45
10	Los Angeles	United States	240
11	Austin	United States	207.97
12	Palo Alto	United States	158
13	Toronto	Canada	155
14	San Jose	United States	128
15	Fremont	United States	124
16	Ann Arbor	United States	122
17	Seattle	United States	119.5
18	Charlotte	United States	100.19
19	Cambridge	United States	93.45
20	Atlanta	United States	83
21	Salem	United States	63.49
22	South San Francisco	United States	58.93
23	Denver	United States	41.98
24	Yokohama	Japan	17.99
25	Zurich	Switzerland	16.99




For the countries that have revenue, the ranking is as follows:
1	United States	2499
2	Canada	155
3	Germany	56.49
4	Japan	17.99
5	Switzerland	16.99


