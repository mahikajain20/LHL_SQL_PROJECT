# Final-Project-Transforming-and-Analyzing-Data-with-SQL

## Project/Goals

- To apply the knowledge learned over the past six weeks and before about SQL and data science
 
- To reinforce and practice SQL coding and complex queries, preparing me for any technical tests or implementation in the field
 
- Creating an ERD, understanding relationships between variables, trying to gain insights from data that I have no background about
 
- Data exploration and brainstorming practice, looking at data from different angles
 
- Succinctly presenting my work and using visual data instead of code, communicating findings to an audience
 
- To learn from my peers’ approaches towards the data test, their project flow and processes
 
- To learn from mistakes and how my coding practices for SQL can improve

## Process
### Downloading the files onto my computer
### Learning how to use Git and GitBash through lecture and mentors
### Creating a database e-commerce in PGAdmin
This involved looking at the table structure preliminarily in excel and then creating the table structure using CREATE statements with the correct data types for the variables. Initially, I had to go back and forth as there were some variables with values that were out of bounds (impossible values). This is also part of the data cleaning. Then I copied the csv data into the sql tables and checked if all the tables were imported properly.

### Data cleaning 
This seemed like the obvious step after importing the data into pgadmin. However, after looking at the amount of null values that were present in the tables, it seemed like a daunting task. It would have also taken a really long time and decision-making to figure out what to remove and what to keep. I consulted with a mentor who directed me to try out the query questions.
This advice was really helpful as it helped me interact with the data more instead of viewing it as a giant hurdle in my way. Therefore, I planned out the answers to my queries and after several combinations, I cleaned the data as I went and redid my queries with a cleaner version. This involved changing the data formatting, removing some inconsistent product IDs that were not adding much value to the table, removing duplicates and correcting string formatting. The *cleaning_data.md* file elaborates further on this.

### Question Queries
The results of the queries using the cleaner tables were generated using concepts learnt in the past 6 weeks such as Common Table Expressions (CTEs), functions, WHERE statements etc. Some questions required some judgement on my part, which helped me reflect on how during data analysis, the scientist or analyst must go through a mental tug or war between various avenues they could take to answer a question and get familiar with the process of how they come to a decision based on the goal of the project. I catered the answers of my questions based on the goal of the project to help an e-commerce business. The results were copied onto the *starting_with_questions.md* file which can be found in this repository.

### Exploring Data
Now, I had freedom to explore the aspects of the data I was interested in from the perspective of a consultant looking to help an e-commerce business website. I decided to first brainstorm some ideas and then visualise if those answers were within the scope of information that I had or my coding skills. I came up with 5 questions ranging from supply chain optimisation to time-series analysis of revenue to explore the dataset I was provided with. The results of this analysis can be found in *starting_with_data.md* attached in the repository. 

### Quality Assurance (QA) Process
Identified more risk areas in the dataset and expanding on the QA process. I found that data cleaning was closely linked to QA and it can even be considered a part of QA. More information about the QA process can be found in *QA.md.*

### Entity Relationship Diagram (ERD)
I identified the primary keys in the earlier steps, however, it was challenging to identify what foreign keys I could reference as there were duplicates present or unequal information in the tables. Therefore, I kept most of my structure simple and created an Entity Relationship Diagram with the established relationships. This is also attached as *schema_png* in the repository.

### Presentation
Lastly, I created a presentation slide deck to present my process and findings to my peers and instructors. I received good feedback from the instructor.
*Link:* https://docs.google.com/presentation/d/1tznUrAiHbifKxkaPIQZDjTwOJ--1rCVnjpVP3lFAIHo/edit#slide=id.g2c06900f62a_0_192

## Results
There were a lot of insights derived from this data, which can be seen in the files attached. The United States is the biggest customer base and loves water bottles and apparel. It generates the most revenue for this business. 
I will focus on my data exploration here, as it is unique to my project. I am also adding how these insights can help a business. 

A quick summary includes: 

### Year with the Highest Revenue:

The year 2016 recorded the highest revenue, totaling $2,403,797. This insight can guide strategic planning and resource allocation for future years. More data should be collected for 2017 August onwards.

### Most Common Channels for Website Access:

Identified the most common channels through which visitors accessed the website, providing insights into user engagement and traffic sources. Organic Search was the most common channel of engagement. This information informs marketing strategies and channel optimization efforts.

### Revenue Trend Over Time:

Analyzed the trend of revenue over time using a moving average approach, providing insights into revenue fluctuations and long-term patterns. This analysis can help identify seasonal trends, market dynamics, and potential areas for revenue growth or optimization.

### Correlation between Sentiment Score and Total Orders:

No significant correlation was found between the sentiment score of products and the total number of orders, suggesting that customer sentiment may not strongly influence purchasing behavior. This highlights the need for further exploration into factors driving customer purchase decisions.

### Average Restocking Lead Time and Products Above Average:

The average restocking lead time was 12 (days?) and identified products with restocking lead times above the average were found, facilitating inventory management and restocking optimization. It was great that only a few products were out of stock, however, factors that were increasing restocking times for popular products such as the Home Cameras can be investigated to increase sales.

## Challenges 

1. The all_sessions table presented challenges due to numerous missing values and duplicated data, making it difficult for analysis. The lack of a clear primary key in this table increased the potential for redundant data. 
2. A lot of the queries dealt with geographical analysis of the data, however, for a lot of the products, the information on the countries and the cities was incomplete. 
3. One of the biggest challenges was understanding what the tables and the data actually mean. There was dire need for metadata or a data description file.
4. Lot of mismatch between tables, however, cross validation was difficult due to lack of proper primary or foreign keys
5. Understanding quality assurance techniques and explanation of the process
6. The ERD diagram and creating relationships between certain IDs such as visit_id, as there were duplicates. It may not be the most complex as I was unsure on how to navigate this.
7. Working with errors and understanding how the results were wrong, even if there was not an error was difficult without visualization. 

## Future Goals

**Data Standardization:** Implement processes to normalize text fields, unify date formats, and standardize categorical values. For example, standardizing product names and categories across related tables.

**Better Data Cleaning:** Refining duplicate detection algorithms, improving missing data imputation techniques, and implementing advanced outlier detection methods. For example, prevent duplicate visit IDs and orderIDs to make sure there is a primary key in every table to detect duplicates for other variables more easily, leading to a better analysis.

**Information about the Variables:** Gaining metadata including descriptions, data types, allowable values, and any constraints or dependencies. Document the meaning and significance of each column in your dataset, including their sources and potential data transformations. This documentation can help users interpret the data correctly and avoid misinterpretations.

**Visualization:** Employing visualization techniques such as interactive dashboards, heatmaps, and trend analysis can help uncover patterns, trends, and anomalies more effectively. Visual representations can aid in diagnosing issues and communicating findings to stakeholders.

**Query Performance:** Optimizing query performance involves tuning database configurations, indexing strategies, and query structures to improve efficiency and reduce latency. Implementing techniques such as query optimization, indexing, and caching can significantly enhance performance for both ad-hoc and repetitive queries.

