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







