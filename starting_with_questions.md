Answer the following questions and provide the SQL queries used to find the answer.

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**


SQL Queries:

-- Create a CTE with the necessary information needed to answer the question

-- TABLE: all_sessions (temp table tbl)

-- COLUMNS: fullvisitorid, country, city, totaltransactionrevenue

-- CITY: Since there are many records that have 'not available in demo dataset', use a CASE WHEN statement to update the records to NULL

-- Use the SUM aggregate function and GROUP BY function to see the total transaction revenue per city/country

-- Order by total transaction revenue in descending so the highest is listed at the top

'''

SQL
CREATE TEMP TABLE tbl AS
SELECT fullvisitorid, country, city, totaltransactionrevenue, date, productsku, productquantity, productprice, v2productname, v2productcategory, ROW_NUMBER () OVER(PARTITION BY fullvisitorid, country, city ORDER BY fullvisitorid) AS rownum
FROM all_sessions

DELETE FROM tbl
WHERE rownum > 1

WITH transactions AS
(
SELECT 
	fullvisitorid, 
	country, 
	CASE WHEN city = 'not available in demo dataset' THEN NULL 
	ELSE city END AS city, 	
	totaltransactionrevenue 
FROM tbl 
WHERE totaltransactionrevenue IS NOT NULL) 
-- 77 rows 
SELECT 
	country, 
	city, 
	SUM(totaltransactionrevenue) AS totaltransactionrevenue_region 
FROM transactions 
WHERE city IS NOT NULL 
GROUP BY country, city 
ORDER BY totaltransactionrevenue_region DESC

'''

Answer:

The cities that have the highest level of transaction revenue are San Francisco, Sunnyvale and Atlanta, which are all located in the United States.

The countries with the highest level of transaction revenue are United States, Israel, and Australia.


**Question 2: What is the average number of products ordered from visitors in each city and country?**


SQL Queries:

-- Create a CTE with the necessary information needed to answer the question from the all_sessions table and the products table

-- To find the average number of products ordered from visitors in each city

-- Had to remove the null values as there are many records that do not include the city

-- Join the CTE with the products table 

-- Also decided to round the average values to 2 decimals and remove the average values that equal to 0 in order to keep it as clean as possible

'''

SQL
WITH cleanedup_allsessions AS
(SELECT
	fullvisitorid,
	CASE 
 	WHEN country = '(not set)' 
 	THEN NULL
	ELSE country
	END AS country, 
	CASE 
 	WHEN city = 'not available in demo dataset' THEN NULL
 	WHEN city = '(not set)' THEN NULL
	ELSE city
	END AS city, 
	productSKU
 FROM tbl --temp table
)
SELECT 
	alls.country, 
	alls.city, 
	ROUND(CAST(AVG(p.orderedquantity) AS numeric), 2) AS average_ordered_products
FROM cleanedup_allsessions alls
JOIN products p ON alls.productSKU = p.sku
WHERE alls.country IS NOT NULL 
AND alls.city IS NOT NULL
GROUP BY alls.country, alls.city
HAVING AVG(p.orderedquantity) > 0
ORDER BY country, city

'''

-- To find the average number of products ordered from visitors in each country

'''

SQL
WITH cleanedup_allsessions AS
(SELECT
	fullvisitorid,
	CASE WHEN country = '(not set)' THEN NULL
	ELSE country
	END AS country, 
	CASE 
 	WHEN city = 'not available in demo dataset' THEN NULL
 	WHEN city = '(not set)' THEN NULL
	ELSE city
	END AS city, 
	productSKU
 FROM tbl --temp table
)
SELECT 
	alls.country, 
	ROUND(CAST(AVG(p.orderedquantity) AS numeric), 2) AS average_ordered_products
FROM cleanedup_allsessions alls
JOIN products p ON alls.productSKU = p.sku
WHERE alls.country IS NOT NULL 
AND alls.city IS NOT NULL
GROUP BY alls.country
HAVING AVG(p.orderedquantity) > 0
ORDER BY country

'''

Answer:

There are 257 different cities listed in the results, so the query gives us 257 different averages. For example, in Buenos Aires, Argentina, the average number of products ordered is 488.40. In Rosario, Argentina, the average number of products ordered is 433.00. In Santa Fe, Argentina, the average number of products ordered is 3682.00.

There are 59 different countries listed in the results, so the query gives us 59 different averages. For example, the average number of products ordered in all of Argentina is 936.71. In Australia, the average number of products ordered is 515.21. In Austria, the average number of products ordered is 245.33.



**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:

-- To clean up the data, we'll re-categorize the product categories to more general categories and remove the ones that are uncertain since some subcategories exist in multiple general categories

-- In the CASE WHEN statement, I used the UPPER function to make sure the data catches as much as possible since it is case-sensitive

-- Removed the values where the total number of ordered quantity is 0

-- After, I found the rank per category for the number of products ordered for each city and country using the RANK and PARTITION BY functions

-- In order to eliminate the number of decimal places, the ROUND function was used to make the data cleaner and more presentable and the orderedquantity column needed to be changed to a numeric value in order to compute the SUM

-- Joined the temporary table to the products table to get the orderedquantity


-- This query is for each city

'''

SQL
WITH category AS
(
SELECT 
	fullvisitorid, 
	CASE WHEN country = '(not set)' THEN NULL
	ELSE country
	END AS country, 
	CASE 
 	WHEN city = '(not set)' THEN NULL
 	WHEN city = 'not available in demo dataset' THEN NULL
	ELSE city
	END AS city, 
	productsku, 
	CASE 
	WHEN UPPER(v2productcategory) LIKE '%ACCESSORIES%' THEN 'Accessories'
	WHEN UPPER(v2productcategory) LIKE '%APPAREL%' THEN 'Apparel'
	WHEN UPPER(v2productcategory) LIKE '%BAGS%' THEN 'Bags'
	WHEN UPPER(v2productcategory) LIKE '%BRANDS%' THEN 'Brands'
	WHEN UPPER(v2productcategory) LIKE '%DRINKWARE%' THEN 'Drinkware'
	WHEN UPPER(v2productcategory) LIKE '%ELECTRONICS%' THEN 'Electronics'
	WHEN UPPER(v2productcategory) LIKE '%KIDS%' THEN 'Kids'
	WHEN UPPER(v2productcategory) LIKE '%LIFESTYLE%' THEN 'Lifestyle'
	WHEN UPPER(v2productcategory) LIKE '%LIMITED SUPPLY%' THEN 'Limited Supply'
	WHEN UPPER(v2productcategory) LIKE '%NEST%' THEN 'Nest'
	WHEN UPPER(v2productcategory) LIKE '%OFFICE%' THEN 'Office'
	WHEN UPPER(v2productcategory) LIKE '%SALE%' THEN 'Sale'
	WHEN UPPER(v2productcategory) LIKE '%SHOP BY BRAND%' THEN 'Shop by Brand'
	WHEN UPPER(v2productcategory) LIKE '%WAZE%' THEN 'Shop by Brand'
	WHEN UPPER(v2productcategory) LIKE '%YOUTUBE%' THEN 'Shop by Brand'
	ELSE NULL
	END AS product_category
FROM tbl --temp table
)
SELECT 
	c.product_category,
	c.country, 
	c.city,
	ROUND(CAST(SUM(p.orderedquantity) AS numeric), 2) AS ordered_products,
	RANK() OVER(PARTITION BY c.product_category ORDER BY SUM(p.orderedquantity) DESC) AS rank
FROM category c
JOIN products p ON c.productSKU = p.sku
WHERE c.country IS NOT NULL 
AND c.city IS NOT NULL
AND c.product_category IS NOT NULL
GROUP BY c.product_category, c.country, c.city
HAVING SUM(p.orderedquantity) > 0
ORDER BY c.product_category, rank, c.country, c.city

'''

-- This query would be to find the country with the highest number of products and see if there is a pattern.

'''

SQL
WITH category AS
(
SELECT 
	fullvisitorid, 
	CASE WHEN country = '(not set)' THEN NULL
	ELSE country
	END AS country, 
	CASE 
 	WHEN city = '(not set)' THEN NULL
 	WHEN city = 'not available in demo dataset' THEN NULL
	ELSE city
	END AS city, 
	productsku, 
	CASE 
	WHEN UPPER(v2productcategory) LIKE '%ACCESSORIES%' THEN 'Accessories'
	WHEN UPPER(v2productcategory) LIKE '%APPAREL%' THEN 'Apparel'
	WHEN UPPER(v2productcategory) LIKE '%BAGS%' THEN 'Bags'
	WHEN UPPER(v2productcategory) LIKE '%BRANDS%' THEN 'Brands'
	WHEN UPPER(v2productcategory) LIKE '%DRINKWARE%' THEN 'Drinkware'
	WHEN UPPER(v2productcategory) LIKE '%ELECTRONICS%' THEN 'Electronics'
	WHEN UPPER(v2productcategory) LIKE '%KIDS%' THEN 'Kids'
	WHEN UPPER(v2productcategory) LIKE '%LIFESTYLE%' THEN 'Lifestyle'
	WHEN UPPER(v2productcategory) LIKE '%LIMITED SUPPLY%' THEN 'Limited Supply'
	WHEN UPPER(v2productcategory) LIKE '%NEST%' THEN 'Nest'
	WHEN UPPER(v2productcategory) LIKE '%OFFICE%' THEN 'Office'
	WHEN UPPER(v2productcategory) LIKE '%SALE%' THEN 'Sale'
	WHEN UPPER(v2productcategory) LIKE '%SHOP BY BRAND%' THEN 'Shop by Brand'
	WHEN UPPER(v2productcategory) LIKE '%WAZE%' THEN 'Shop by Brand'
	WHEN UPPER(v2productcategory) LIKE '%YOUTUBE%' THEN 'Shop by Brand'
	ELSE NULL
	END AS product_category
FROM tbl --temp table
)
SELECT 
	c.product_category,
	c.country, 
	ROUND(CAST(SUM(p.orderedquantity) AS numeric), 2) AS ordered_products,
	RANK() OVER(PARTITION BY c.product_category ORDER BY SUM(p.orderedquantity) DESC) AS rank
FROM category c
JOIN products p ON c.productSKU = p.sku
WHERE c.country IS NOT NULL 
AND c.city IS NOT NULL
AND c.product_category IS NOT NULL
GROUP BY c.product_category, c.country
HAVING SUM(p.orderedquantity) > 0
ORDER BY c.product_category, rank, c.country

'''

Answer:

Based on the ranks, we notice that in most of the product categories, the number of products ordered are from Mountain View and New York in the United States, with the exception of San Francisco ranking 1st in Brands and 2nd in Drinkware. For the products under Lifestyle, Santiago, Chile is ranked 2nd.

The country with the highest number of products ordered in all the product categories is the United States. After United States, the next few countries with the highest number of products ordered varies between Canada, India, United Kingdom, and a few others but these four countries are usually at the top, based on their rank.

Most of our results will be skewed towards the United States, since more than half of our records are from the United States.


**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:

-- Created 3 CTEs for each table to have a more concise and cleaner dataset to work with when comparing them together (using tables all_sessions, analytics and products)

-- For the analytics table, we needed the fullvisitorid, the units_sold, the unit_price (divided by 1,000,000) and the calculated revenue

-- For the all_sessions table, we needed the fullvisitorid, country, city, and productSKU

-- For the products table, we needed the sku and a cleaned up version of the product name (to clean up the leading and trailing spaces and make sure the names are consistent)

-- Then we would include the rank for products sold from highest total revenue and partition it per city/country


-- The first query is to find the top-selling products in each city and the second query is for each country

'''

SQL
WITH 
testdata AS
(
SELECT 
	fullvisitorid, 
	units_sold, 
	ROUND(CAST(unit_price / 1000000 AS numeric), 2) AS single_unitprice, 
	ROUND(CAST((units_sold * unit_price / 1000000) AS numeric),2) AS revenue
FROM analytics
WHERE units_sold > 0
),
allsessions_new AS
(
SELECT
	fullvisitorid,
	CASE WHEN country = '(not set)' 
 	THEN NULL
	ELSE country
	END AS country, 
	CASE 
	WHEN city = 'not available in demo dataset' THEN NULL
	WHEN city = '(not set)' THEN NULL
	ELSE city
	END AS city, 
	productSKU
 FROM tbl --temp table
),
product_info AS
(
SELECT
	sku,
	TRIM(' ' FROM name) AS clean_name
FROM products
)
SELECT 
	alls.country,
	alls.city,
	p.sku, 
	p.clean_name, 
	SUM(t.revenue) AS total,
	RANK () OVER(PARTITION BY alls.country ORDER BY SUM(t.revenue) DESC) AS rank 
FROM allsessions_new alls
JOIN testdata t ON t.fullvisitorid = alls.fullvisitorid
JOIN product_info p ON alls.productsku = p.sku
WHERE alls.country IS NOT NULL AND alls.city IS NOT NULL
GROUP BY alls.country, alls.city, p.sku, p.clean_name
HAVING SUM(revenue) > 0
ORDER BY alls.country, alls.city, rank

'''

-- QUERY 2

'''

SQL
WITH 
testdata AS
(
SELECT 
	fullvisitorid, 
	units_sold, 
	ROUND(CAST(unit_price / 1000000 AS numeric), 2) AS single_unitprice, 
	ROUND(CAST((units_sold * unit_price / 1000000) AS numeric),2) AS revenue
FROM analytics
WHERE units_sold > 0
),
allsessions_new AS
(
SELECT
	fullvisitorid,
	CASE WHEN country = '(not set)' 
 	THEN NULL
	ELSE country
	END AS country, 
	CASE 
	WHEN city = 'not available in demo dataset' THEN NULL
	WHEN city = '(not set)' THEN NULL
	ELSE city
	END AS city, 
	productSKU
 FROM tbl --temp table
),
product_info AS
(
SELECT
	sku,
	TRIM(' ' FROM name) AS clean_name
FROM products
)
SELECT 
	alls.country,
	p.sku,
	p.clean_name, 
	SUM(t.revenue) AS total,
	RANK () OVER(PARTITION BY alls.country ORDER BY SUM(t.revenue) DESC) AS rank 
FROM allsessions_new alls
JOIN testdata t ON t.fullvisitorid = alls.fullvisitorid
JOIN product_info p ON alls.productsku = p.sku
WHERE alls.country IS NOT NULL AND alls.city IS NOT NULL
GROUP BY alls.country, p.sku, p.clean_name
HAVING SUM(revenue) > 0
ORDER BY alls.country, rank

'''

Answer:

Looking at all the top rank in each country, the top-selling product in each city/country is as followed:

Australia: Men's Vintage Tank
- Sydney: Men's Vintage Tank

Canada: Men's Vintage Tank
- Kitchener: 5-Panel Cap
- Toronto: Men's Vintage Tank

Chile: Sport Bag
- Santiago: Sport Bag

Colombia: Men's 3/4 Sleeve Henley
- Bogota: Men's 3/4 Sleeve Henley

Finland: Android Onesie Gold
- Helsinki: Android Onesie Gold

France: Android Lunch Kit
- Courbevoie: Android Wool Heather Cap Heather/Black
- Paris: Android Lunch Kit

Germany: 22 oz Bottle Infuser
- Berlin: Doodle Decal
- Hamburg: 22 oz Bottle Infuser
- Munich: Men's 100% Cotton Short Sleeve Hero Tee Red

Hong Kong: Device Stand
- Hong Kong: Device Stand

India: Bongo Cupholder Bluetooth Speaker
- Hyderabad: Bongo Cupholder Bluetooth Speaker

Indonesia: Windup Android
- Jakarta: Windup Android

Ireland: Laptop Backpack
- Dublin: Laptop Backpack

Israel: Men's Vintage Tank
- Tel Aviv-Yafo: Men's Vintage Tank

Japan: 5-Panel Snapback Cap
- Minato: 5-Panel Snapback Cap
- Osaka: Infant Short Sleeve Tee Red
- Yokohama: Rucksack

Singapore: Men's Short Sleeve Performance Badge Tee Navy
- Singapore: Men's Short Sleeve Performance Badge Tee Navy

South Korea: Dress Socks
- Seoul: Dress Socks

Spain: Protect Smoke + CO White Wired Alarm-USA
- Madrid: Protect Smoke + CO White Wired Alarm-USA

Sweden: Android Rise 14 oz Mug
- Stockholm: Android Rise 14 oz Mug

Switzerland: Rocket Flashlight
- Zurich: Rocket Flashlight

Taiwan: Spiral Notebook and Pen Set
- Zhongli District: Spiral Notebook and Pen Set

Thailand: 26 oz Double Wall Insulated Bottle
- Bangkok: 26 oz Double Wall Insulated Bottle

United Kingdom: Tote Bag
- London: Tote Bag

United States: Baby on Board Window Decal
- Ann Arbor: Men's Vintage Tank
- Atlanta: Onesie Green
- Austin: RFID Journal
- Cambridge: Screen Cleaning Cloth
- Charlotte: Lunch Bag
- Chicago: Learning Thermostat 3rd Gen-USA - Copper
- Cupertino: Protect Smoke + CO White Wired Alarm-USA
- Dallas: Leatherette Notebook Combo
- Denver: Twill Cap
- Detroit: 22 oz Water Bottle
- Fremont: Protect Smoke + CO White Battery Alarm-USA
- Houston: Sunglasses
- Irvine: Men's Vintage Badge Tee Sage
- Kirkland: Women's Long Sleeve Tee Lavender
- Los Angeles: Cam Indoor Security Camera - USA
- Milpitas: Women's Short Sleeve Hero Dark Grey
- Mountain View: Baby on Board Window Decal
- Nashville: Learning Thermostat 3rd Gen-USA - Stainless Steel
- New York: Collapsible Shopping Bag
- Palo Alto: Cam Outdoor Security Camera - USA
- Paris: Women's Quilted Insulated Vest Black ***
- Phoenix: Men's Short Sleeve Hero Tee Heather
- Pittsburgh: Hard Cover Journal
- Salem: Men's Zip Hoodie
- San Bruno: 22 oz Bottle Infuser
- San Diego: Doodle Decal
- San Francisco: Learning Thermostat 3rd Gen-USA - Stainless Steel
- San Jose: 22 oz Android Bottle
- Santa Clara: Clip-on Compact Charger
- Santa Monica: Slim Utility Travel Bag
- Seattle: Cam Indoor Security Camera - USA
- South San Francisco: Rucksack ***
- Sunnyvale: Men's Convertible Vest-Jacket Pewter
- Washington: Bib White
- Montevideo: Vintage Henley Grey/Black


There isn't anything drastic we can tell from the products sold besides some products in particular being the top-selling product in multiple countries such as the Men's Vintage Tank for Australia, Canada, and Israel. Also, there are a couple cities in the United States that can be renamed such as Paris since it falls under France and South San Francisco can fall under San Francisco.



**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:

-- Create a CTE joining the all_sessions and analytics tables together (both tables cleaned up a bit)

-- Using a subquery to get the total revenue for all the data that was used (without the null countries/cities), we can get a percentage of the revenue generated from each city/country

-- Reordered the results by descending order from highest revenue to lowest

'''

SQL
WITH joineddata AS
(
SELECT 
	alls.fullvisitorid, 
	CASE WHEN alls.country = '(not set)' THEN NULL 
	ELSE alls.country 
	END AS country, 
	CASE WHEN alls.city = 'not available in demo dataset' THEN NULL 
	WHEN alls.city = '(not set)' THEN NULL 		
	ELSE alls.city 
	END AS city,
	a.units_sold, 
	ROUND(CAST(a.unit_price / 1000000 AS numeric), 2) AS unitprice, 
	ROUND(CAST((a.units_sold * a.unit_price / 1000000) AS numeric),2) AS revenue
FROM tbl alls --temp table
JOIN analytics a
USING (fullvisitorid)
WHERE a.units_sold > 0
)
SELECT 
	country,
	city,
	SUM(revenue) / 
	(SELECT SUM(revenue) AS totalrevenue
FROM joineddata
WHERE country IS NOT NULL
AND city IS NOT NULL) * 100  AS percentage
FROM joineddata jd
WHERE country IS NOT NULL
AND city IS NOT NULL
GROUP BY country, city
ORDER BY SUM(revenue) DESC

'''

-- QUERY 2 (only country)

'''

SQL
WITH joineddata AS
(
SELECT 
	alls.fullvisitorid, 
	CASE WHEN alls.country = '(not set)' THEN NULL 
	ELSE alls.country 
	END AS country, 
	CASE WHEN alls.city = 'not available in demo dataset' THEN NULL 
	WHEN alls.city = '(not set)' THEN NULL 		
	ELSE alls.city 
	END AS city,
	a.units_sold, 
	ROUND(CAST(a.unit_price / 1000000 AS numeric), 2) AS unitprice, 
	ROUND(CAST((a.units_sold * a.unit_price / 1000000) AS numeric),2) AS revenue
FROM tbl alls --temp table
JOIN analytics a
USING (fullvisitorid)
WHERE a.units_sold > 0
)
SELECT 
	country,
	SUM(revenue) / 
	(SELECT SUM(revenue) AS totalrevenue
FROM joineddata
WHERE country IS NOT NULL
AND city IS NOT NULL) * 100  AS percentage
FROM joineddata
WHERE country IS NOT NULL
AND city IS NOT NULL
GROUP BY country
ORDER BY SUM(revenue) DESC

'''

Answer:

Between the cities, we see that majority of the revenue comes from Mountain View at 70.16%. The next highest on the list is from San Bruno at 15.79%, then New York at 4.80%, which are all in the United States.

Between just the countries, we see that 99.68% of the revenue comes from the United States, which makes sense since most of the records come from the United States as well. The next highest percentage comes from Canada at 0.11%, then Hong Kong at 0.07%.

If we are only looking at the countries, it would be hard to summarize the data unless we only want data specifically for the United States since almost all the revenue is coming from the United States.
Even when looking at the cities in the United States, there is still a good chunk of the revenue that comes from Mountain View.
