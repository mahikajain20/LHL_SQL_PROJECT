What are your risk areas? Identify and describe them.

The all_sessions table presented challenges due to numerous missing values and duplicated data, making it difficult for analysis. Despite efforts to clean the data and write queries, ensuring proper removal of duplicates was challenging due to the absence of a primary key, even a composite one. The lack of a clear primary key increased the potential for redundant data. For example, there were duplicated visitor id’s which does make sense as the same visitors can visit the site multiple times a day. However, what didn’t make sense was that there were multiple visit id’s which should be unique as each visit is unique.

A lot of the queries dealt with geographical analysis of the data, however, for a lot of the products, the information on the countries and the cities was incomplete. This affected the analysis significantly as sometimes an unknown city would have the most amounts of products ordered, however, it was impossible to determine which one. In some analyses, I decided to remove them and in some I decided to keep them, depending on what I was trying to draw from the query. Grouping by country was generally easier, as the data was more complete on that end. But there were some weird city country combinations such as Los Angeles in Australia which are risking the integrity of the data.

The specificity of the categories was a risk area as it was had two sides. One side was that we can gain great detail on the type of categories that users were ordering, but in the other hand, too much detail can distract the analysis and it was hard to understand which category was really the more commonly ordered one. I decided to only truncate the strings if the category column, however, I did not further condense it as I wanted to give more weight to the specificity of the categories.

One of the biggest risk areas was understanding what the tables and the data actually mean. Usually I’ve worked with datasets where I have more background or information about the variables so it is easier to build relationships and draw conclusions for the data analysis. However, this time, the interpretations drawn from this data have to be read with caution due to the large number of missing information, in the dataset and about the dataset. More information about the data collection process or a data description can reduce this risk area.

There is a skill issue present here as well, which is a risk area in itself. I am not informed about advanced data cleaning techniques yet or quality assurance techniques, which limits the QA process of this project and I hope I can reduce this risk area by furthering my knowledge in this area.

Most of the QA queries I have used are present in the cleaning_data.md file, however, I have added some more here.

QA Process:
Describe your QA process and include the SQL queries used to execute it.

- Using total_ordered and ordered_quantity: Identifying match or mismatch 

```sql
SELECT s.product_SKU,
       s.total_ordered AS sales_total_ordered,
       p.ordered_quantity AS product_ordered_quantity,
       CASE WHEN s.total_ordered = p.ordered_quantity THEN 'Match' ELSE 'Mismatch' END AS quantity_match_status
FROM sales_by_sku s
JOIN products p ON s.product_SKU = p.SKU;
```

There were a lot of mismatches so these columns cannot be used interchangeably. 

- No primary key for all_sessions or analytics

The query aims to determine if any combination of attributes in the analytics table could serve as a primary key. However, the absence of groups with duplicate counts greater than one suggests that no single attribute or combination of attributes uniquely identifies each record in the table, indicating a lack of a primary key.

```sql
SELECT visit_id,
full_visitor_id,
visit_start_time,
visit_date,
time_on_site,
COUNT(*) AS duplicates
FROM analytics
GROUP BY visit_id, full_visitor_id, visit_start_time, visit_date, time_on_site
HAVING COUNT(*) > 1
```

The result was that there were no such columns that could serve as primary key here. I tried the same thing with the all_sessions2 table where I thought product_sku, full_visitor_id or visit_id could have been primary keys.

```sql
SELECT full_visitor_id,
COUNT(*) AS duplicates
FROM all_sessions2
GROUP BY full_visitor_id
HAVING COUNT(*) > 1
```

```sql
SELECT visit_id,
COUNT(*) AS duplicates
FROM all_sessions2
GROUP BY visit_id
HAVING COUNT(*) > 1
```

```sql
SELECT product_sku,
COUNT(*) AS duplicates
FROM all_sessions2
GROUP BY product_sku
HAVING COUNT(*) > 1
```

All of these had duplicates so it was not possible to have a unique identifier other than the index of each row itself.

Identifying mismatch between tables, pertaining to the same type of variable. This is affecting data integrity, especially for crucial variables like product name. The first query selects distinct product names from the all_sessions table that are not found in the products table. These are products that have been recorded in the sessions but are not listed in the main products database. The second query selects distinct product names from the products table that are not found in the all_sessions table. These are products listed in the main products database but have not been recorded in any session.

E.g.

```sql
SELECT DISTINCT(v2_product_name)
FROM all_sessions
WHERE v2_product_name NOT IN (SELECT productname
FROM products)
ORDER BY v2_product_name -- 379 distinct products in all_sessions table that are not included in the products table
```

```sql
SELECT DISTINCT(product_name)
FROM products
WHERE product_name NOT IN (SELECT v2_product_name
FROM all_sessions)
ORDER BY product_name -- 221 distinct products in products table that are not included in the all_sessions table
```

Lastly, before exploration of data, to identify which columns would be not beneficial to work with due to a large number of Null Values, I ran these queries.

```sql
-- Explore non-null values for total_transaction_revenue
SELECT total_transaction_revenue
FROM all_sessions2
WHERE total_transaction_revenue IS NOT NULL
```

```sql
-- Explore non-null values for transactions in all_sessions table
SELECT transactions
FROM all_sessions2
WHERE transactions IS NOT NULL
```

```sql
-- Explore non-null values for product_quantity (does it mean product quantity ordered or available?)
SELECT product_quantity
FROM all_sessions2
WHERE product_quantity
IS NOT NULL
```

```sql
-- Explore non-null values for product_price
SELECT product_price
FROM all_sessions2
WHERE product_price IS NOT NULL
```

```sql
-- Explore non-null values for product_revenue in all_sessions table
SELECT product_revenue
FROM all_sessions2
WHERE product_revenue IS NOT NULL -- 3 rows
```

```sql
-- Non-null values for product_sku in all_sessions table
SELECT DISTINCT(product_sku)
FROM all_sessions2
WHERE product_sku IS NOT NULL
```

```sql
-- Item_revenue in all_sessions table
SELECT item_revenue
FROM all_sessions2
WHERE item_revenue IS NOT NULL
-- 0 rows so column deleted
```

```sql
-- transaction_revenue in all_sessions table
SELECT transaction_revenue
FROM all_sessions2
WHERE transaction_revenue IS NOT NULL
```

```sql
-- Transaction ID
SELECT transaction_id
FROM all_sessions2
WHERE transaction_id IS NOT NULL 
```

This shows that some columns are absolutely redundant such as transactions, transaction_id, transaction_revenue, item revenue, product_revenue. The rest had some information, however, it was still limited and not matching other tables. This is the biggest risk area for this QA process as the data is very noisy and incomplete.
I generated the average product revenue on my own using CTEs, as shown in other files. This was as none of the revenue columns had enough values for analysis of product revenue.

I did a similar analysis for the analytics table and products table. It was not as involved in the data analysis so I did not put the queries here.

### Summary 
For the QA process, if I had more time, I would have cross tabulated the values in all tables to check if the values were consistent. I would have also done deeper data range validation to see what outliers were present in the data.

To conclude, throughout the data analysis process, several key lessons were learned that can inform future analyses. One crucial takeaway is the importance of robust data cleaning and preprocessing techniques to address missing values, duplicates, and inconsistencies effectively. While efforts were made to clean the data, the lack of a clear primary key and incomplete geographic information posed significant challenges. This underscores the need for comprehensive data quality checks and documentation to ensure data integrity and reliability.

Moreover, the analysis revealed the value of exploring data quality issues early in the process to identify potential pitfalls and limitations. By conducting exploratory queries to examine non-null values, duplicates, and discrepancies between tables, insights were gained into the underlying data structure and its implications for analysis.

Looking ahead, it's recommended to invest in further data validation and quality assurance measures, including cross-referencing data from multiple sources and conducting thorough data profiling exercises. This could involve implementing automated data validation scripts, establishing data quality metrics, and enhancing documentation practices to track data lineage and transformations more effectively. Additionally, incorporating more descriptive metadata and data dictionaries can provide valuable context for understanding the dataset and its variables.
