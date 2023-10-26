What are your risk areas? Identify and describe them.

There are many null values under the city column in comparison to the country column. Removing the null records could potentially change our results drastically.
There are many records that have mismatched countries and cities. For example, there is a record of Los Angeles (city) being in Australia (country) or New York (city) in Canada (country) when Los Angeles and New York are both in the United States.
These inconsistencies would alter the final result.

QA Process:
Describe your QA process and include the SQL queries used to execute it.

- Removing duplicates
- Filtering out the null values
- Matching the tables with those who have similar information such as sales_by_sku and sales_report (productSKU and the total_ordered)
