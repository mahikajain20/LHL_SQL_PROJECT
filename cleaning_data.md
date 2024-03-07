What issues will you address by cleaning the data?

1. **Missing Values**: Identifying and handling missing data, which might involve filling in missing values with a suitable default or removing records with missing data. In this case, from the limited knowledge that I had, I chose to forego the null values and get rid of columns that were redundant in the database. 

2. **Duplicate Records**: Detecting and removing duplicate entries to prevent inaccurate results and to also ensure that there are no duplicates for columns that are ID / Primary Key columns. 

3. **Removing Impossible Values and Managing Outliers**: Identifying and correcting data entry errors or implausible values, such as negative ages. Also, identifying extreme values that are past the normal bounds of a column’s data range so that the results of the analysis are not skewed. 

4. **Dealing with Inconsistent Formatting**: This step involves correcting formatting or certain columns such as data, ID, mixed capitalization, inconsistent formatting across a dataset for the same thing etc. In this case, the data format was very strange so it had to be properly casted to make sure that it could be understood and worked with. Then the productSKU ID column had some inconsistencies in the form that the SKU ID and so on.

5. **Data Types**: This is an extension of the formatting aspect but it deals with particularly the type of variable that SQL thinks is in a column. This step involves ensuring that each column's data type is appropriate for its content, such as converting strings to numeric types if necessary. 

6. **Strings**: After a skim over the tables, this step involves identifying columns that have unnecessary additions in the text data. This was seen in some columns of the all sessions table. 


Queries:
Below, provide the SQL queries you used to clean your data.

1. **Converting unit_price/product_price**
A water bottle should not cost millions of dollars so therefore, the price of the products have to be converted. Here, it seems plausible to divide the price by a million (1000000). Therefore, I created a function for it as I had to use it multiple times and I did not want to hardcode it multiple times. 

This is the code I used to create the function.
```sql
CREATE OR REPLACE FUNCTION convert_price_mil(price double precision)
RETURNS double precision AS $$
BEGIN
    -- Convert the price to the desired format
    RETURN price / 1000000.0;
END;
$$ LANGUAGE plpgsql;

```
This was used inside CTEs when calculating revenue in the questions that I had to answer or created based on the data. (q5 for example) 

2. **Duplicate Primary Keys**
Primary keys are primary keys because they are unique. However, to check if that is indeed the case, I used a query to check if there are duplicate values in certain ID columns such as full_visitorid, product_sku, userid. 
This is one example.

```sql
SELECT
    count(full_visitorid),
    full_visitorid
FROM
    all_sessions2
GROUP BY
    full_visitorid
HAVING
    COUNT(*) > 1;
```

In this example, the full_visitorid column indeed have 868 rows with duplicates. Therefore, I checked if visit_id had duplicates as the visitor could be the same, but each visit is unique. 

```sql
SELECT visit_id, COUNT(*)
FROM all_sessions2
GROUP BY visit_id
HAVING COUNT(*) > 1;
```
There are 553 rows with duplicates here as well, which does not say great things about the data that we have. 

```sql 
SELECT sku, COUNT(*)
FROM products
GROUP BY sku
HAVING COUNT(*) > 1;
```
This yielded in 0 results, which was a big relief. However, it did have multiple occurrences in the all_sessions database. 

3. **Null Values**

From first glance at the tables, I could see that there were plenty of null values. 
I used this query just to check how many null values were present in this particular column, and I edited it to check for other columns as well. 

```sql
SELECT
    COUNT(*) AS counting
FROM
    analytics
WHERE
    user_id IS NULL;
```

This query helped me see that 4301122 of the visitors were not a user, or that the system did not register that number of userIDs, which makes it really difficult to keep track of user transactions. I made sure to use WHERE IS NOT NULL statements in my queries to get accurate results. 

4. **Formatting Date**

The all_sessions table had a date column with just integer values. This needed to be dealt with so I converted the column to date and fixed the format. 

```sql
/* Convert session_date data type from integer to date format */
ALTER TABLE
    all_sessions2
ALTER COLUMN
    session_date TYPE DATE USING to_date(to_char(session_date, '99999999'), 'YYYYMMDD');

/* Verify */
SELECT session_date FROM all_sessions2
```

5. **Removing useless spaces and characters from strings**

```sql
/* Remove useless characters from the product names */
UPDATE all_sessions2
SET v2_product_name = REGEXP_REPLACE(v2_product_name, '[^a-zA-Z0-9\s]', '', 'g');
/* Verify */
SELECT v2_product_name FROM all_sessions2
```

This query removes any characters from the product_name column that are not alphanumeric or whitespace characters. The 'g' flag ensures that all matching substrings in each row are replaced, not just the first one.

6. **Inconsistent Data**

When I was looking at the product_sku column, I realized some of the rows have a different format than the usual “GG…” one so I performed a query to check which rows are being violated.

First: 

```sql
SELECT sku
FROM products
WHERE sku NOT LIKE 'GG%';
```
There were 170 entries where the pattern was not followed. When I checked how these entries looked like in table as a whole, all of them had an ordered_quantity of 0. Therefore, they were not relevant to my analysis and I chose to remove them from the data. In the real world, I would have done this after consulting with someone. However, as it was not really adding anything to my analysis here, I decided to get rid of them. 

```sql
DELETE FROM products
WHERE sku NOT LIKE 'GG%';

DELETE FROM all_sessions2
WHERE product_SKU NOT LIKE 'GG%';

DELETE FROM sales_by_sku
WHERE product_SKU NOT LIKE 'GG%';

DELETE FROM sales_report
WHERE product_SKU NOT LIKE 'GG%';
```

Now, this query below returned no results. 

```sql
SELECT product_SKU
FROM sales_report
WHERE product_SKU NOT LIKE 'GG%';
```



