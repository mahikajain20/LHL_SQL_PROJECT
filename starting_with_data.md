**Question 1:** 
What year and what month in that year made the most revenue? 

**SQL Queries:**
```sql
WITH avg_data AS (
    SELECT 
        a.product_SKU,
        AVG(a.product_price / 1000000) AS avg_price,
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
), revenue_summary AS (
    SELECT 
        product_SKU,
        avg_price,
        avg_total_ordered,
        (avg_price * avg_total_ordered) AS revenue
    FROM 
        avg_data
)
--Using CTE, extracting year from session dates and getting the highest one
SELECT 
 	EXTRACT(YEAR FROM a.session_date) AS year,
	SUM(r.revenue) AS total_revenue
FROM 
    all_sessions2 a
JOIN 
    revenue_summary r ON a.product_SKU = r.product_SKU
GROUP BY
	year
ORDER BY 
    total_revenue DESC;
```
Answer of this: 

![Query 1 Data ](https://github.com/mahikajain20/LHL_SQL_PROJECT/assets/131741978/3369c85d-89aa-494d-a423-19b3bbb6d9b6)

This could either due to sales or as data collection was stopped in August 2017, it had an equal distribution. So now let's look at what month/s make the most sales. 

```sql
WITH avg_data AS (
    SELECT 
        a.product_SKU,
        AVG(a.product_price / 1000000) AS avg_price,
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
), revenue_summary AS (
    SELECT 
        product_SKU,
        avg_price,
        avg_total_ordered,
        (avg_price * avg_total_ordered) AS revenue
    FROM 
        avg_data
)
--Using CTE, extracting year from session dates and getting the highest one
SELECT 
 	EXTRACT(MONTH FROM a.session_date) AS month,
	EXTRACT(YEAR FROM a.session_date) AS year,
	SUM(r.revenue) AS total_revenue
FROM 
    all_sessions2 a
JOIN 
    revenue_summary r ON a.product_SKU = r.product_SKU
GROUP BY
	month, 
	year
ORDER BY 
    total_revenue DESC;
```

This leads to the output: 

![Query 1(2) data](https://github.com/mahikajain20/LHL_SQL_PROJECT/assets/131741978/eb7a92f1-8d9f-4cf0-8f1b-01ee725497b9)


**Answer:** 
2016 made the most sales, particularly due to the holiday season as seen by December having the most sales in 2016. 2017 January was the second, as the holiday season continues. 
This could suggest the business to increase engagement most during holiday season, have sales and plan products accordingly. 


**Question 2:** 
What are the most common channels through which visitors access the website? 

**SQL Queries:**

```sql
SELECT
    channel_grouping,
    COUNT(*) AS visit_count
FROM
    all_sessions2
GROUP BY
    channel_grouping
ORDER BY
    visit_count DESC;
```
I used the analytics table here, and not the allsessions table because the analytics table had more data on Channel Grouping and Engagement. a larger sample size could provide more insight into what channels users use to access the ecommerce website. 


**Answer:**

![Query 2 Data ](https://github.com/mahikajain20/LHL_SQL_PROJECT/assets/131741978/4bcecbde-b5ab-416d-895f-1943b88d8af7)


**Question 3:**
How does the revenue change over time?

**SQL Queries:**

```sql
WITH revenue_trend AS (
    SELECT
        session_date,
        AVG(rs.revenue) OVER (ORDER BY session_date ROWS BETWEEN 30 PRECEDING AND CURRENT ROW) AS moving_avg_revenue
    FROM
        all_sessions2 a
    JOIN
        revenue_summary_view rs ON a.product_SKU = rs.product_SKU
)

SELECT
    rt.session_date,
    rt.moving_avg_revenue
FROM
    revenue_trend rt
ORDER BY rt.session_date
```
This table output may not be the most clear at the moment, however, using visualization techniques or filtering for particular time periods, it can provide important information to see the trajectory for revenue for the ecommerce website. 

**Answer:**

![image](https://github.com/mahikajain20/LHL_SQL_PROJECT/assets/131741978/a2b7a77f-31ec-425f-ac2d-5b36e3b0fdef)

![image](https://github.com/mahikajain20/LHL_SQL_PROJECT/assets/131741978/86a6f18d-5066-4ebb-8d76-5f85c1065952)


**Question 4:** 
Is there a correlation between the number of times a product is ordered and its sentiment score?

**SQL Queries:**
To check this, I used the CORR function in SQL along with a CTE. 

```sql
WITH sentiment_sales AS (
    SELECT
        p.product_name,
        p.sentiment_score,
        s.total_ordered
    FROM
        products p
    JOIN
        sales_report s ON p.SKU = s.product_SKU
    WHERE
        p.sentiment_score IS NOT NULL
        AND s.total_ordered IS NOT NULL AND s.total_ordered != 0
)
SELECT
    CORR(sentiment_score, total_ordered) AS correlation_coefficient
FROM
    sentiment_sales
```
**Answer:**

![image](https://github.com/mahikajain20/LHL_SQL_PROJECT/assets/131741978/61510c87-766b-4eaf-8278-ed6773f82f50)

This is a very low value, which suggests that there is basically no correlation between these two things. It would be helpful if I knew a bit more about how this sentiment score was calculated.

**Question 5:** Are there any restocking_lead_times above average (of the lead times), and if so, for what products? (Supply chain optimization)

Restocking lead time refers to the amount of time it takes for a business to replenish its inventory after it has been sold or depleted. It encompasses the entire process from recognizing the need to restock a product to receiving the new inventory. The average restocking lead time for products provides insights into how efficiently a business manages its inventory and how quickly it can respond to changes in demand. I wanted to run this query to see if any products took longer than average to restock as there were some products that were out of stock in the datatset. 

**SQL Queries:**
```sql
--Products out of stock
SELECT 
    sku,
    product_name,
    stock_level,
	restocking_lead_time
FROM 
    products
WHERE 
    stock_level <= 0;
-- Average restocking time 
WITH avg_restocking_lead_time AS (
    SELECT AVG(restocking_lead_time) AS average_restocking_lead_time
    FROM products
)
-- Products that have above average restocking lead times
SELECT 
    product_name,
	restocking_lead_time
FROM 
    products
CROSS JOIN 
    avg_restocking_lead_time
WHERE 
    restocking_lead_time > average_restocking_lead_time
ORDER BY restocking_lead_time DESC;
```

**Answer:**
![image](https://github.com/mahikajain20/LHL_SQL_PROJECT/assets/131741978/442d8621-47be-4be9-8439-5895b6171a98)

There were more products than I expected as there were only 20 products out of stock. One of the products with the highest restocking time also has high sales, which is interesting. Further inquiry can be done if customers buy it off as soon as it is in stock and what is the time period or pattern in that. 
