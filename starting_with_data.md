Question 1: How many unique visitors were there by month?

SQL Queries:
```SQL
WITH visitors_by_month AS
(
SELECT--DISTINCT is used up front to ensure any given fullvisitorid cannot be combined with the same year and month combination twice.
	DISTINCT EXTRACT(YEAR FROM sessiondate) AS sessionyear--we group and order data using this, to keep months separate (otherwise 2016 August and 2017 August would be lumped together)
	, (EXTRACT(MONTH FROM sessiondate)) AS sessionmonth--this month value is used for grouping and ordering the data. we use the created namedmonth column below to display our data.
	, COUNT(fullvisitorid) OVER (--because of our DISTINCT in the SELECT statement this gives us a count of DISTINCT fullvisitorid values.
		PARTITION BY--both year and month to keep every single month separated without having to resort to more logic statements.
			EXTRACT(YEAR FROM sessiondate)
			, EXTRACT(MONTH FROM sessiondate)
		ORDER BY
			EXTRACT(YEAR FROM sessiondate)
			, EXTRACT(MONTH FROM sessiondate)
	) AS uniquevisitors
	, TO_CHAR(sessiondate,'Month') AS namedmonth--this becomes sessionmonth when displayed.
FROM
	all_sessions
ORDER BY--ensuring month and year are in order.
	EXTRACT(YEAR FROM sessiondate)
	, EXTRACT(MONTH FROM sessiondate)
)

SELECT
	sessionyear
	, namedmonth as sessionmonth
	, uniquevisitors
FROM
	visitors_by_month
```
	


Answer: 
sessionyear	sessionmonth	uniquevisitors
2016	August   	2035
2016	September	1624
2016	October  	1008
2016	November 	1122
2016	December 	1215
2017	January  	1069
2017	February 	1098
2017	March    	1099
2017	April    	1105
2017	May      	1255
2017	June     	1176
2017	July     	1289
2017	August   	39



Question 2: What cities and countries have the highest amount of hits?
```SQL
SQL Queries:
SELECT
	alls.city
	, alls.country
	, SUM(vh.totalhits) AS sumhits
FROM
	all_sessions alls
JOIN
	visit_hits vh--a table created using data queried from the larger dataset online. contains hits by fullvisitorid, visit combo.
	USING
		(fullvisitorid)
WHERE
	city <> 'Not Available in Demo Dataset'--removing improperly labled cities.
GROUP BY--country is here to account for cities with the same name in different countries.
	city
	, country
ORDER BY
	sumhits DESC--starts with highest value
LIMIT 10


--COUNTRY
SELECT
	alls.country
	, SUM(vh.totalhits) AS sumhits
FROM
	all_sessions alls
JOIN
	visit_hits vh--a table created using data queried from the larger dataset online. contains hits by fullvisitorid, visit combo.
	USING
		(fullvisitorid)
GROUP BY
	 country
ORDER BY
	sumhits DESC--starts with highest value
LIMIT 10
```
Answer:

city	country	sumhits
Salem	United States	40170
Mountain View	United States	31704
New York	United States	17775
Sunnyvale	United States	7703
San Francisco	United States	7389
Chicago	United States	6932
San Jose	United States	6481
Montreal	Canada	5107
Ann Arbor	United States	3093
Austin	United States	3010


country	sumhits
United States	260990
Canada	18758
Japan	3939
United Kingdom	3596
India	3590
Taiwan	3081
Singapore	2997
France	2011
Germany	1883
Australia	1615


Question 3: What was the average amount of hits across sessions that viewed certain product categories?
```SQL
SQL Queries:
SELECT
	alls.v2productcategory
	, AVG(vh.totalhits)::NUMERIC(10,2) AS avghits--making the average more readable by cutting down decimal places. you can't have a part of a hit, but this can help conceptualize the averages better.
FROM
	all_sessions alls
JOIN
	visit_hits vh--a table created using data queried from the larger dataset online. contains hits by fullvisitorid, visit combo.
	USING
		(visitid)--to account for specific sessions. fullvisitorids can have multiple visitids, so the data wouldn't be the same.
GROUP BY
	alls.v2productcategory--want to know the average for product category
```
Answer:
v2productcategory	avghits
${escCatTitle}	9
(not set)	4.66
Apparel	16.2
Bags	14.5
Bottles/	1
Drinkware	5.67
Electronics	21.5
Headgear	8
Home/Accessories/	5.77
Home/Accessories/Drinkware/	5.01
Home/Accessories/Fun/	6.61
Home/Accessories/Housewares/	7.02
Home/Accessories/Pet/	6.42
Home/Accessories/Sports & Fitness/	7
Home/Accessories/Stickers/	4.93
Home/Apparel/	5.22
Home/Apparel/Headgear/	4.39
Home/Apparel/Kid's/	5.86
Home/Apparel/Kid's/Kid's-Infant/	6.05
Home/Apparel/Kid's/Kid's-Toddler/	5.3
Home/Apparel/Kid's/Kids-Youth/	5.05
Home/Apparel/Men's/	5.63
Home/Apparel/Men's/Men's-Outerwear/	4.95
Home/Apparel/Men's/Men's-Performance Wear/	6.23
Home/Apparel/Men's/Men's-T-Shirts/	4.08
Home/Apparel/Women's/	5.77
Home/Apparel/Women's/Women's-Outerwear/	6.73
Home/Apparel/Women's/Women's-Performance Wear/	5.55
Home/Apparel/Women's/Women's-T-Shirts/	5.33
Home/Bags/	5.07
Home/Bags/Backpacks/	6.19
Home/Bags/More Bags/	7.21
Home/Bags/Shopping and Totes/	6.24
Home/Brands/	3.67
Home/Brands/Android/	5.85
Home/Brands/YouTube/	4.6
Home/Clearance Sale/	4
Home/Drinkware/	4.88
Home/Drinkware/Mugs and Cups/	6.11
Home/Drinkware/Water Bottles and Tumblers/	5.97
Home/Electronics/	5.77
Home/Electronics/Accessories/Drinkware/	6
Home/Electronics/Audio/	5.76
Home/Electronics/Electronics Accessories/	5.7
Home/Electronics/Flashlights/	6.65
Home/Electronics/Power/	7.47
Home/Fruit Games/	5.79
Home/Fun/	2.67
Home/Gift Cards/	3.3
Home/Kids/	6
Home/Lifestyle/	6.66
Home/Lifestyle/Fun/	1
Home/Limited Supply/	6.25
Home/Limited Supply/Bags/	6.01
Home/Limited Supply/Bags/Backpacks/	3
Home/Nest/Nest-USA/	4.13
Home/Office/	6.43
Home/Office/Notebooks & Journals/	5.18
Home/Office/Office Other/	6
Home/Office/Writing Instruments/	6.56
Home/Shop by Brand/	6.5
Home/Shop by Brand/Android/	5.17
Home/Shop by Brand/Google/	5.78
Home/Shop by Brand/Waze/	7.33
Home/Shop by Brand/YouTube/	2.87
Home/Spring Sale!/	5.22
Lifestyle	13
Lifestyle/	1.67
Office	8
Waze	15
Wearables/Men's T-Shirts/	4.4
YouTube	2



Question 4: Is there any pattern between the total hits and total unique visitors for each city/country?

SQL Queries:
```SQL
--city
SELECT
	alls.city
	, alls.country--accounting for cities of same name and different country.
	, SUM(vh.totalhits) AS sumhits--GROUP BY is to have this value be by the city.
	, COUNT(DISTINCT(alls.fullvisitorid)) AS uniquevisitors--GROUP BY is to have this value be by the city.
	, (SUM(vh.totalhits) / COUNT(DISTINCT(alls.fullvisitorid)))::NUMERIC(10,2) AS hitsvsvisitors--a comparison to help tell if there is a pattern.
FROM
	all_sessions alls
JOIN
	visit_hits vh--a table created using data queried from the larger dataset online. contains hits by fullvisitorid, visit combo.
	USING
		(visitid)--to account for specific sessions. fullvisitorids can have multiple visitids, so the data wouldn't be the same.
GROUP BY--accounting for cities of same name and different country. sumhits, uniquevisitors, and hitsvsvisitors are aggregated according to this.
	alls.city
	, alls.country
ORDER BY--want to see by numerical values to better answer the question. DESC to start ordering from highest value.
	uniquevisitors DESC
	, sumhits DESC


--country
SELECT
	alls.country
	, SUM(vh.totalhits) AS sumhits--GROUP BY is to have this value be by the city.
	, COUNT(DISTINCT(alls.fullvisitorid)) AS uniquevisitors--GROUP BY is to have this value be by the city.
	, (SUM(vh.totalhits) / COUNT(DISTINCT(alls.fullvisitorid)))::NUMERIC(10,2) AS hitsvsvisitors--a comparison to help tell if there is a pattern.
FROM
	all_sessions alls
JOIN
	visit_hits vh--a table created using data queried from the larger dataset online. contains hits by fullvisitorid, visit combo.
	USING
		(visitid)--to account for specific sessions. fullvisitorids can have multiple visitids, so the data wouldn't be the same.
GROUP BY
	alls.country
ORDER BY --want to see by numerical values to better answer the question. DESC to start ordering from highest value.
	uniquevisitors DESC
	, sumhits DESC
```
Answer:

There doesn't appear to be a strong pattern between total hits and unique visitors per city. Some people generate more hits than others. 
And some locations generated less hits than others despite having more uniquevisitors.  Like London United Kingdom having 172 less hits than Chicago United States while having 16 more unique visitors.

The same can be said for the pattern between total hits and unique visitors per country.

Question 5: Is there any pattern between the total hits on sessions where products are viewed and how many of those products are ordered?

SQL Queries:
```SQL
SELECT--just the info we want.
	name
	, sumhits AS totalhits
	, sumordered AS totalordered
FROM--ordering the totals we got so we can filter out the highest ones per product.
	(
SELECT
	*
	, ROW_NUMBER()
		OVER(
			PARTITION BY
				sku--if multiple different products have the same name we don't want to lump them together.
				, name
			ORDER BY
				sumhits DESC--highest value first.
			) AS orderedsumhits
	, ROW_NUMBER()
		OVER(
			PARTITION BY
				sku--if multiple different products have the same name we don't want to lump them together.
				, name
			ORDER BY
				sumordered DESC--highest value first.
			) AS orderedsumordered
FROM--getting the totals according to specific product.
(
		SELECT
			DISTINCT p.sku--included to ensure different products with the same name are treated as different products.
			, p.name
			, SUM(vh.totalhits) OVER (
				PARTITION BY
					p.sku--in case multiple different products have the same name.
					, p.name
				ORDER BY
					p.sku
					, p.name) AS sumhits
			, SUM(sbs.total_ordered) OVER (
				PARTITION BY
					p.sku--in case multiple different products have the same name.
					, p.name
				ORDER BY
					p.sku 
					, p.name) AS sumordered
		FROM
			visit_hits vh--a table created using data queried from the larger dataset online. contains hits by fullvisitorid, visit combo.
		JOIN
			all_sessions alls
			ON--some of the tables have different names for the columns. using ON for all joins here to keep code consistent.
				vh.visitid = alls.visitid
		JOIN
			products p
			ON
				alls.productsku = p.sku
		JOIN
			sales_by_sku sbs--more sku values in here than sales_report.
			ON
				p.sku = sbs.productsku
		GROUP BY--what totalhits and total_ordered are calculated according to.
			p.sku
			, p.name
			, vh.totalhits
			, sbs.total_ordered
		ORDER BY--an easy way to compare the hits on a products with how many orders a product had. can see how the order values change by hits. DESC to start from the highest value.
			sumhits DESC
			, sumordered DESC	
	)

)
WHERE
	orderedsumhits = 1
AND
	orderedsumordered = 1
ORDER BY
	sumhits DESC
	, sumordered DESC
```
Answer:
There doesn't seem to be a pattern between the total hits on sessions where products are viewed and how many of the product are ordered. The product with the most hits (  Alpine Style Backpack, with 265 hits) has 60 ordered. The product with the second most hits ( Men's Short Sleeve Hero Tee Black, with 246 hits) has over twice the amount ordered at 152 ordered. There are some products with around the same amount ordered as there are hits ( Android Sticker Sheet Ultra Removable with 198 and 189 respectively) And there are some products with significantly more ordered than there are hits (Sport Bag with 1190 and 175 respectively)
