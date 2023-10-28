What issues will you address by cleaning the data?

After exploring the data, I notice that there are many null values and some inconsistencies in the data.

Because I do not want to modify the raw data, I decided to create CTEs with the necessary information in my queries when trying to find solutions to the different questions.

-- Checking for duplicates

In the all_sessions table, since there are many records that do not have a country or city, I will rename them as NULL.
Since the questions asked for both city and country, I did two separate queries (one for country/city and the other for just the country).

In both the products and the sales_report tables, I noticed that in the product name, there are many records that have a leading space so I wanted to trim all the leading and trailing spaces in case the same product show up but one has a leading space and one doesn't so they're counted separately.

In the analytics table, I noticed there are discrepancies in the categories and I wanted to categorize them more generally so we can put all the Apparel under the same category so we have around 15 different categories instead of having over 50 different categories.

Also in the analytics table, whenever using the unit_price column, I made sure to divide by 1,000,000 to get the single unit_price.


Queries:
Below, provide the SQL queries you used to clean your data.

**-- GETTING RID OF DUPLICATES**



**-- COUNTRY / CITY (using as a CTE) (NULL VALUES)**

SELECT
	fullvisitorid,
	CASE WHEN country = '(not set)' 
 	THEN NULL
	ELSE country
	END AS country, 
	CASE 
 	WHEN city = 'not available in demo dataset' 
 	WHEN city = '(not set)' 
	THEN NULL
	ELSE city
	END AS city, 
	productSKU
 FROM all_sessions

-- When using the country/city, in the WHERE statement, I would make sure that the country and the city is not NULL.

**-- TRIMMING LEADING/TRAILING SPACES**

SELECT
	sku,
	TRIM(' ' FROM name) AS clean_name
FROM products

**-- RE-CATEGORIZING THE PRODUCT SUBCATEGORIES**

SELECT 
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

-- Since there were some subcategories that belong in multiple general categories, I went with most of the ones that come after Home/
I also used the UPPERCASE statement in case there were inconsistencies with the capitalization of the text.

**-- UPDATING THE UNIT PRICE**

SELECT 
	fullvisitorid, 
	units_sold, 
	ROUND(CAST(unit_price / 1000000 AS numeric), 2) AS single_unitprice, 
	ROUND(CAST((units_sold * unit_price / 1000000) AS numeric),2) AS revenue
FROM analytics

-- I also changed the data type from bigint to numeric so I can use the ROUND function to get rid of the multiple decimal places for cleaner looking data.
