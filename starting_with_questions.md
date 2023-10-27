Answer the following questions and provide the SQL queries used to find the answer.

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**


SQL Queries:

-- Create a CTE with the necessary information needed to answer the question

-- TABLE: all_sessions 

-- COLUMNS: fullvisitorid, country, city, totaltransactionrevenue

-- CITY: Since there are many records that have 'not available in demo dataset', use a CASE WHEN statement to update the records to NULL


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

-- Also decided to round the average values to 2 decimals and remove the average values that equal to 0 in order to keep it as clean as possible


WITH cleanedup_allsessions AS
(SELECT
	fullvisitorid,
	CASE WHEN country = '(not set)' 
 	THEN NULL
	ELSE country
	END AS country, 
	CASE WHEN city = 'not available in demo dataset' OR city = '(not set)' 
	THEN NULL
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
	CASE WHEN country = '(not set)' 
 	THEN NULL
	ELSE country
	END AS country, 
	CASE WHEN city = 'not available in demo dataset' OR city = '(not set)' 
	THEN NULL
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

-- To clean up the data, we'll re-categorize the product categories to more general categories
-- Removed the values where the total number of ordered quantity is 0


WITH category AS
(
SELECT 
	fullvisitorid, 
	CASE WHEN country = '(not set)' 
	THEN NULL
	ELSE country
	END AS country, 
	CASE WHEN city = '(not set)' OR city = 'not available in demo dataset' 
	THEN NULL
	ELSE city
	END AS city, 
	productsku, 
	CASE 
	WHEN v2productcategory LIKE '%Accessories%' THEN 'Accessories'
	WHEN v2productcategory LIKE '%Apparel%' THEN 'Apparel'
	WHEN v2productcategory LIKE '%Bags%' THEN 'Bags'
	WHEN v2productcategory LIKE '%Brands%' THEN 'Brands'
	WHEN v2productcategory LIKE '%Drinkware%' THEN 'Drinkware'
	WHEN v2productcategory LIKE '%Electronics%' THEN 'Electronics'
	WHEN v2productcategory LIKE '%Kids%' THEN 'Kids'
	WHEN v2productcategory LIKE '%Lifestyle%' THEN 'Lifestyle'
	WHEN v2productcategory LIKE '%Limited Supply%' THEN 'Limited Supply'
	WHEN v2productcategory LIKE '%Nest%' THEN 'Nest'
	WHEN v2productcategory LIKE '%Office%' THEN 'Office'
	WHEN v2productcategory LIKE '%Sale%' THEN 'Sale'
	WHEN v2productcategory LIKE '%Shop by Brand%' THEN 'Shop by Brand'
	WHEN v2productcategory LIKE '%Waze%' THEN 'Shop by Brand'
	WHEN v2productcategory LIKE '%YouTube%' THEN 'Shop by Brand'
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

Answer:

Based on the ranks, we notice that in most of the product categories, the number of products ordered are from Mountain View and New York.

However, most of our results will be skewed towards the United States, since more than half of our records are from the United States.




**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:



Answer:





**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:



Answer:







