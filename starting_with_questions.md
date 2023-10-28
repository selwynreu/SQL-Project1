Answer the following questions and provide the SQL queries used to find the answer.

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**


SQL Queries:

-- Create a CTE with the necessary information needed to answer the question

-- TABLE: all_sessions 

-- COLUMNS: fullvisitorid, country, city, totaltransactionrevenue

-- CITY: Since there are many records that have 'not available in demo dataset', use a CASE WHEN statement to update the records to NULL

-- Use the SUM aggregate function and GROUP BY function to see the total transaction revenue per city/country

-- Order by total transaction revenue in descending so the highest is listed at the top


WITH transactions AS

(SELECT 
	fullvisitorid, 
	country, 
 	CASE WHEN
 	city = 'not available in demo dataset' 	THEN NULL
	ELSE city
 	END AS city, 
	totaltransactionrevenue
FROM all_sessions
WHERE totaltransactionrevenue IS NOT NULL)
-- 81 rows
SELECT country, city, SUM(totaltransactionrevenue) AS totaltransactionrevenue_region
FROM transactions
WHERE city IS NOT NULL
GROUP BY country, city
ORDER BY totaltransactionrevenue_region DESC

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
 FROM all_sessions
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


-- To find the average number of products ordered from visitors in each country


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
 FROM all_sessions
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


Answer:

There are 257 different cities listed in the results, so the query gives us 257 different averages. For example, in Buenos Aires, Argentina, the average number of products ordered is 434.50.
In Rosario, Argentina, the average number of products ordered is 433.00. In Santa Fe, Argentina, the average number of products ordered is 1932.50.

There are 59 different countries listed in the results, so the query gives us 59 different averages. For example, the average number of products ordered in all of Argentina is 767.22. In Australia, the average number of products ordered is 505.45. In Austria, the average number of products ordered is 228.50.




**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:

-- To clean up the data, we'll re-categorize the product categories to more general categories and remove the ones that are uncertain since some subcategories exist in multiple general categories

-- In the CASE WHEN statement, I used the UPPER function to make sure the data catches as much as possible since it is case-sensitive

-- Removed the values where the total number of ordered quantity is 0

-- After, I found the rank per category for the number of products ordered for each city and country using the RANK and PARTITION BY functions

-- In order to eliminate the number of decimal places, the ROUND function was used to make the data cleaner and more presentable and the orderedquantity column needed to be changed to a numeric value in order to compute the SUM

-- Joined the temporary table to the products table to get the orderedquantity


-- This query is for each city

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
FROM all_sessions
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


-- This query would be to find the country with the highest number of products and see if there is a pattern.


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
FROM all_sessions
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


Answer:

Based on the ranks, we notice that in most of the product categories, the number of products ordered are from Mountain View and New York in the United States. The country with the highest number of products ordered in all the product categories is the United States. After United States, the next few countries with the highest number of products ordered varies between Canada, India, United Kingdom, and a few others but these four countries are usually at the top, based on their rank.

However, most of our results will be skewed towards the United States, since more than half of our records are from the United States.




**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:



Answer:





**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:



Answer:







