Answer the following questions and provide the SQL queries used to find the answer.

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**

As the question mentions transaction revenues, this led me to believe that I should use the total_transaction_revenue column provided in the all_sessions table. Even though it is missing for several rows as transactions may have not taken place there or entered in, I ran a query to get the answers with the information that was provided. 

**SQL Queries:** 
 
```sql
SELECT country, city, SUM(total_transaction_revenue)/1000000 AS total_revenue
FROM all_sessions2
WHERE total_transaction_revenue IS NOT NULL
GROUP BY country, city
ORDER BY total_revenue DESC;
```

**Answer:**  An unnamed city in the US has the highest level of transaction revenue, then San Francisco, Sunnyvale and so on. Most of the transaction revenues come from the United States. 
![Query 1 ](https://github.com/mahikajain20/LHL_SQL_PROJECT/assets/131741978/8e4fe3f5-986a-4953-8358-5acfc36055fe)



**Question 2: What is the average number of products ordered from visitors in each city and country?**

The average number of products ordered from visitors is calculated here using the total number of products per country and dividing that by the number of unique visitors on the site per country. I understand this could have done in many ways, however I felt that this way was a better way to understand the traffic on the site. 

**SQL Queries:**

```sql
/* Data Overview */
SELECT * FROM sales_by_sku;
SELECT * FROM all_sessions2;

/* Query to join the tables, get number of unique visitors using CTE, total no. of products ordered and then average products ordered */

WITH visitor_counts AS (
    SELECT
        country,
        city,
        COUNT(DISTINCT full_visitorid) AS unique_visitors
    FROM
        all_sessions2
    WHERE
        country != '(not set)'
    GROUP BY
        country, city
)
SELECT
    vc.country,
    vc.city,
    COUNT(s.total_ordered) AS total_products_ordered,
    ROUND(CAST(COUNT(s.total_ordered) AS NUMERIC) / vc.unique_visitors, 2) AS avg_products_ordered
FROM
    visitor_counts vc
JOIN
    all_sessions2 a ON vc.country = a.country AND vc.city = a.city
JOIN
    sales_by_sku s ON a.product_sku = s.product_sku
GROUP BY
    vc.country, vc.city, vc.unique_visitors;
```

**Answer:**

![Query 2](https://github.com/mahikajain20/LHL_SQL_PROJECT/assets/131741978/d752b675-98cc-4524-aff6-97ece4858307)


**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**

This geographical analysis can provide important information to a company regarding what kind of products are ordered in what regions. 

SQL Queries:

This query below focuses on the country as it led to more understandable results due to a lot of missing values in the city column.

```sql
-- Create a temporary table 
CREATE TEMPORARY TABLE temp_results AS
SELECT a.country, a.v2_Product_Category, SUM(p.total_ordered) AS total_quantity_ordered
FROM all_sessions2 a
JOIN sales_by_sku p ON a.product_SKU = p.product_SKU
GROUP BY a.country, a.v2_Product_Category
HAVING SUM(p.total_ordered) != 0 AND SUM(p.total_ordered) IS NOT NULL
ORDER BY total_quantity_ordered DESC;

-- Update the v2_Product_Category column in the temporary table for clarity 
UPDATE temp_results
SET v2_Product_Category = REPLACE(v2_Product_Category, 'Home/', '')
WHERE v2_Product_Category LIKE 'Home/%';

-- View the updated results from the temporary table
SELECT * FROM temp_results
WHERE country != '(not set)'
ORDER BY country, total_quantity_ordered DESC;
```
This query below also includes the city information. 

```sql
-- Create a temporary table 
CREATE TEMPORARY TABLE temp_results1 AS
SELECT 
    a.country, 
    a.city,
    a.v2_product_category, 
    SUM(p.total_ordered) AS total_quantity_ordered
FROM 
    all_sessions2 a
JOIN 
    sales_by_sku p ON a.product_SKU = p.product_SKU
WHERE 
    a.country != '(not set)' 
    AND a.city NOT LIKE '%not%'
GROUP BY 
    a.country, 
    a.city,
    a.v2_Product_Category
HAVING 
    SUM(p.total_ordered) != 0 AND SUM(p.total_ordered) IS NOT NULL
ORDER BY 
    a.country, 
    a.city, 
    total_quantity_ordered DESC;

-- Update the v2_Product_Category column in the temporary table for clarity 
UPDATE temp_results1
SET v2_product_category = REPLACE(v2_product_category, 'Home/', '')
WHERE v2_product_category LIKE 'Home/%';

-- View the updated results from the temporary table
SELECT * FROM temp_results1
ORDER BY country, city, total_quantity_ordered DESC;
```

Answer: MountainView in the US uses a lot of SmartHome products (nest) and Office supplies, it had the highest quantity ordered. 
The highest orders are from the US which include drinkware, office supplies, lifestyle etc. 

![Query 3](https://github.com/mahikajain20/LHL_SQL_PROJECT/assets/131741978/90fb3bdc-f4c7-4bb9-b1d7-e11230673c64)


**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**

**SQL Queries:**

For this, I used a CTE and Window Functions to get the top-selling product in each country and city.
The first query does not remove the cities which are not available, as it gives information about what product sells a lot in that country. 

```sql 
WITH top_products_per_city_country AS (
    SELECT 
        a.country, 
        a.city,
        sr.product_name,
        SUM(s.total_ordered) AS total_ordered,
        DENSE_RANK() OVER (PARTITION BY a.country, a.city ORDER BY SUM(s.total_ordered) DESC) AS rank
    FROM 
        all_sessions2 a
    JOIN 
        sales_by_sku s ON a.product_SKU = s.product_SKU
    JOIN 
        sales_report sr ON s.product_SKU = sr.product_sku
    GROUP BY 
        a.country, 
        a.city, 
        sr.product_name
)
SELECT 
    country, 
    city,
    product_name AS top_selling_product,
    total_ordered
FROM 
    top_products_per_city_country
WHERE 
    rank = 1 AND 
	country != '(not set)'
ORDER by total_ordered DESC;
```

This second query disregards the cities altogether and groups by country to get a more concise description of the top-selling product in each country. 

```sql
WITH top_products_per_country AS (
    SELECT 
        a.country, 
        sr.product_name,
        SUM(s.total_ordered) AS total_ordered,
        DENSE_RANK() OVER (PARTITION BY a.country ORDER BY SUM(s.total_ordered) DESC) AS rank
    FROM 
        all_sessions2 a
    JOIN 
        sales_by_sku s ON a.product_SKU = s.product_SKU
    JOIN 
        sales_report sr ON s.product_SKU = sr.product_sku
    GROUP BY 
        a.country,  
        sr.product_name
)
SELECT 
    country, 
    product_name AS top_selling_product,
    total_ordered
FROM 
    top_products_per_country
WHERE 
    rank = 1 AND 
	country != '(not set)'
ORDER by total_ordered DESC;
```

**Answer:** The top-selling product overall is sold in the United States. It is the Nest® Learning Thermostat 3rd Gen-USA. The other countries vary between apparel, hard cover journal, twill caps, bottle infusers etc. There is not a particular pattern that can be deduced from this information as it is mostly stationary items such as journals, pens, bottles etc. 

Screenshot for the country's top-selling products: 

![Query 4](https://github.com/mahikajain20/LHL_SQL_PROJECT/assets/131741978/b2caf8be-0c2b-462d-ac4b-a1659c99f739)


**Question 5: Can we summarize the impact of revenue generated from each city/country?**

For this question, this query calculates the total revenue generated from product sales in each country. It starts by computing the average price and average total ordered quantity for each product SKU from the all_sessions2 table, using the avg_data Common Table Expression (CTE). Then, it calculates the revenue for each SKU by multiplying the average price by the average total ordered quantity. Next, the query aggregates the revenue values by country, summing up the revenues for all product SKUs associated with each country. It also calculates the proper summary using mean, stddev, maximum revenue generated by a product and minimum. 

**SQL Queries:**

```sql
WITH avg_data AS (
    SELECT 
        a.product_SKU,
        AVG(a.product_price/ 1000000) AS avg_price,
        AVG(s.total_ordered) AS avg_total_ordered
    FROM 
        all_sessions2 a
    JOIN 
        sales_by_sku s ON a.product_SKU = s.product_SKU
    WHERE 
        s.total_ordered > 0 
        AND s.total_ordered IS NOT NULL 
        AND a.product_price IS NOT NULL
    GROUP BY 
        a.product_SKU
)

SELECT 
    a.country,
    CAST(SUM(revenue_summary.revenue) AS NUMERIC(15, 1)) AS total_revenue,
    AVG(revenue_summary.revenue) AS mean_revenue,
    STDDEV(revenue_summary.revenue) AS stddev_revenue,
    MAX(revenue_summary.revenue) AS max_revenue,
    MIN(revenue_summary.revenue) AS min_revenue
FROM (
    SELECT 
        product_SKU,
        avg_price,
        avg_total_ordered,
        (avg_price * avg_total_ordered) AS revenue
    FROM 
        avg_data
) AS revenue_summary
JOIN 
    all_sessions2 a ON revenue_summary.product_SKU = a.product_SKU
WHERE 
    a.country != '(not set)'
GROUP BY 
    a.country
ORDER BY 
    total_revenue DESC;
```


**Answer:**
Countries that have prosperous economies have seen to generate greater total revenue. This suggests that further analysis could be done on how developed and underdeveloped countries differ in their ecommerce use. 

![query 5 (2)](https://github.com/mahikajain20/LHL_SQL_PROJECT/assets/131741978/3a5bfbd9-0fd1-465c-a363-f72591d79989)

![graph_visualiser-1709769922475](https://github.com/mahikajain20/LHL_SQL_PROJECT/assets/131741978/bf707e81-c810-45e6-8c84-4947049d4cec)





