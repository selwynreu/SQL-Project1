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



Answer:





**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:



Answer:





**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:



Answer:





**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:



Answer:







