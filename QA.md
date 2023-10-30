What are your risk areas? Identify and describe them.

There are many null values under the city column in comparison to the country column. Removing the null records could potentially change our results drastically.
There are many records that have mismatched countries and cities. For example, there is a record of Los Angeles (city) being in Australia (country) or New York (city) in Canada (country) when Los Angeles and New York are both in the United States.
These inconsistencies would alter the final result.

QA Process:
Describe your QA process and include the SQL queries used to execute it.

- Removing duplicates by creating a temporary table to not alter the data and also using 

CREATE TEMP TABLE tbl AS
SELECT fullvisitorid, country, city, totaltransactionrevenue, date, productsku, productquantity, productprice, v2productname, v2productcategory, ROW_NUMBER () OVER(PARTITION BY fullvisitorid, country, city ORDER BY fullvisitorid) AS rownum
FROM all_sessions

DELETE FROM tbl
WHERE rownum > 1

- Filtering out the null values as they are incomplete data (using a CTE then filtering out the null values)

WITH qatest AS
(
SELECT 
	fullvisitorid,
	CASE 
	WHEN country = '(not set)' THEN NULL
	ELSE country
	END AS country,
	CASE 
	WHEN city = '(not set)' THEN NULL
	WHEN city = 'not available in demo dataset' THEN NULL
	ELSE city
	END AS city
FROM all_sessions
)
SELECT *
FROM qatest
WHERE country IS NOT NULL
AND city IS NOT NULL
  
- Matching the tables with those who have similar information such as sales_by_sku and sales_report (productSKU and the total_ordered)

SELECT sbs.productsku, sbs.total_ordered, sr.productsku, sr.total_ordered
FROM sales_by_sku sbs
JOIN sales_report sr USING (productsku)
WHERE sbs.total_ordered != sr.total_ordered

- Checking if the currency code is all the same amongst all the records.

SELECT *
FROM all_sessions
WHERE currencycode != 'USD'
