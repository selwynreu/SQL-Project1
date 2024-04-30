Question 1: Compare the revenue in months. Is there a pattern that you noticed in the total revenue?

SQL Queries:

```
WITH revenue1 AS (
    SELECT 
        EXTRACT(YEAR FROM t.date) AS year, 
        EXTRACT(MONTH FROM t.date) AS month, 
        CASE 
            WHEN t.country = '(not set)' THEN NULL 
            ELSE t.country 
        END AS country, 
        CASE 
            WHEN t.city = 'not available in demo dataset' THEN NULL 
            WHEN t.city = '(not set)' THEN NULL 
            ELSE t.city 
        END AS city, 
        a.units_sold, 
        ROUND(CAST(a.unit_price / 1000000 AS numeric), 2) AS unitprice, 
        ROUND(CAST((a.units_sold * a.unit_price / 1000000) AS numeric),2) AS revenue 
    FROM 
        tbl t 
    JOIN 
        analytics a USING (fullvisitorid) 
    WHERE 
        revenue > 0 
)
SELECT 
    year, 
    month, 
    SUM(revenue) AS total 
FROM 
    revenue1 
WHERE 
    country IS NOT NULL 
    AND city IS NOT NULL 
GROUP BY 
    year, 
    month 
ORDER BY 
    year, 
    month, 
    total DESC
```

Answer: 

Based on the results, these are the average revenues for each month:

- August 2016: 402.00
- September 2016: 1292.00
- October 2016: 25.00
- November 2016: 555.00
- December 2016: 2592.00
- January 2017: 378.00
- February 2017: 2262.00
- March 2017: 487.00
- April 2017: 2740.00
- May 2017: 6540.00
- June 2017: 4802.00
- July 2017: 9175.00

Since we only have results for 1 year total, it is hard to forecast future sales precisely, but it does show an upwards trend in total revenue.


Question 2: What are the most ordered product categories?

SQL Queries:

'''

SQL
SELECT 
	CASE 
	WHEN UPPER(t.v2productcategory) LIKE '%ACCESSORIES%' THEN 'Accessories' 
	WHEN UPPER(t.v2productcategory) LIKE '%APPAREL%' THEN 'Apparel' 
	WHEN UPPER(t.v2productcategory) LIKE '%BAGS%' THEN 'Bags' 
	WHEN UPPER(t.v2productcategory) LIKE '%BRANDS%' THEN 'Brands' 
	WHEN UPPER(t.v2productcategory) LIKE '%DRINKWARE%' THEN 'Drinkware' 
	WHEN UPPER(t.v2productcategory) LIKE '%ELECTRONICS%' THEN 'Electronics' 
	WHEN UPPER(t.v2productcategory) LIKE '%KIDS%' THEN 'Kids' 
	WHEN UPPER(t.v2productcategory) LIKE '%LIFESTYLE%' THEN 'Lifestyle' 
	WHEN UPPER(t.v2productcategory) LIKE '%LIMITED SUPPLY%' THEN 'Limited Supply' 
	WHEN UPPER(t.v2productcategory) LIKE '%NEST%' THEN 'Nest' 
	WHEN UPPER(t.v2productcategory) LIKE '%OFFICE%' THEN 'Office' 
	WHEN UPPER(t.v2productcategory) LIKE '%SALE%' THEN 'Sale' 
	WHEN UPPER(t.v2productcategory) LIKE '%SHOP BY BRAND%' THEN 'Shop by Brand' 
	WHEN UPPER(t.v2productcategory) LIKE '%WAZE%' THEN 'Shop by Brand' 
	WHEN UPPER(t.v2productcategory) LIKE '%YOUTUBE%' THEN 'Shop by Brand' 
	ELSE NULL 
	END AS product_category,
	SUM(total_ordered) AS total_sum_ordered
FROM sales_report sr
JOIN tbl t
ON sr.productsku = t.productsku
GROUP BY product_category
ORDER BY total_sum_ordered DESC

'''

Answer:

The product category that has the most ordered products is in the 'Shop by Brand' category with 45778, then Office products with 33804 and then Drinkware products with 27942.


Question 3: Which countries have the highest average time spent on the site by visitors?

SQL Queries:

'''

SQL
SELECT country, AVG(timeonsite) AS average_time
FROM tbl
WHERE timeonsite > 0
GROUP BY country
ORDER BY average_time DESC

'''

Answer:

Based on the results, these are the ten countries with the highest average time spent on site:
1. Nigeria (885.60 seconds)
2. Peru (802.00 seconds)
3. Tunisia (726.00 seconds)
4. Slovenia (696.00 seconds)
5. Kenya (572.33 seconds)
6. El Salvador (568.00 seconds)
7. Trinidad & Tobago (561.00 seconds)
8. Jersey (518.00 seconds)
9. Réunion (465.00 seconds)
10. Morocco (457.17 seconds)

