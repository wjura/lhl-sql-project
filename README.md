# Final-Project-Transforming-and-Analyzing-Data-with-SQL

## Project/Goals
The ultimate goal is to practice SQL skills learned during the first month of the Data Science Bootcamp.

Specific Objectives:
- Loading CSV Files into Database: Efficiently import CSV data into PostgreSQL tables.
- Data Cleaning: Clean and standardize the raw data to ensure accuracy and consistency.
- Transforming and Analyzing Data: Perform data transformations and analyses to extract meaningful insights.
- Developing and Implementing a QA Process: Establish and execute a QA process to maintain data integrity and validate results.

By achieving these objectives, the project aims to reinforce SQL proficiency and provide hands-on experience in managing and analyzing real-world data.

## Process
### Creating a new PostgreSQL database called "ecommerce"

I imported all csv files by creating each table with the below code:

```SQL
-- Creating products table:

CREATE TABLE products (
	sku VARCHAR NOT NULL,
 	product_name VARCHAR,
 	ordered_quantity INT,
  	stock_level INT,
	restocking_lead_time INT,
	sentiment_score DOUBLE PRECISION,
	sentiment_magnitude DOUBLE PRECISION,
	PRIMARY KEY (sku)
);

COPY products(
	sku,
	product_name, 
	ordered_quantity, 
	stock_level,
	restocking_lead_time,
	sentiment_score,
	sentiment_magnitude)
FROM 'D:\LHL\projects\data\products.csv'
DELIMITER ','
CSV HEADER;

-- Creating sales_report table:
CREATE TABLE sales_report (
	product_sku VARCHAR NOT NULL,
	total_ordered INT, 
	product_name VARCHAR, 
	stock_level INT,
	restocking_lead_time INT,
	sentiment_score DOUBLE PRECISION,
	sentiment_magnitude DOUBLE PRECISION,
	ratio DOUBLE PRECISION,
	PRIMARY KEY (sku)
);

COPY sales_report(
	product_sku,
	total_ordered, 
	product_name, 
	stock_level,
	restocking_lead_time,
	sentiment_score,
	sentiment_magnitude,
	ratio)
FROM 'D:\LHL\projects\data\sales_report.csv'
DELIMITER ','
CSV HEADER;

-- Creating sales_by_sku table:
CREATE TABLE sales_by_sku (
	product_sku VARCHAR NOT NULL,
	total_ordered INT,
	PRIMARY KEY (product_sku)
);

COPY sales_by_sku(
	product_sku,
	total_ordered)
FROM 'D:\LHL\projects\data\sales_by_sku.csv'
DELIMITER ','
CSV HEADER;

-- Creating analytics table:
CREATE TABLE analytics(
	visit_number INT,
	visit_id INT,
	visit_start_time INT,
	visit_date DATE,
	full_visitor_id VARCHAR,
	user_id VARCHAR,
	channel_grouping VARCHAR,
	social_engagement_type VARCHAR,
	units_sold INT,
	page_views INT,
	time_on_site INT,
	bounces INT,
	revenue NUMERIC,
	unit_price INT
);

COPY analytics(
	visit_number,
	visit_id,
	visit_start_time,
	visit_date,
	full_visitor_id,
	user_id,
	channel_grouping,
	social_engagement_type,
	units_sold,
	page_views,
	time_on_site,
	bounces,
	revenue,
	unit_price)
FROM 'D:\LHL\projects\data\analytics.csv'
DELIMITER ','
CSV HEADER;

-- Creating all_sessions table:
CREATE TABLE all_sessions(
	full_visitor_id VARCHAR,
	channel_grouping VARCHAR,
	visit_time VARCHAR,
	country VARCHAR,
	city VARCHAR,
	total_transaction_revenue INT,
	transactions INT,
	time_on_site INT,
	page_views INT,
	session_quality_dim VARCHAR,
	visit_date DATE,
	visit_id INT,
	visit_type VARCHAR,
	product_refund_amount INT,
	product_quantity INT,
	product_price INT,
	product_revenue INT,
	product_sku VARCHAR,
	v2product_name VARCHAR,
	v2product_category VARCHAR,
	product_variant VARCHAR,
	currency_code VARCHAR,
	item_quantity NUMERIC,
	item_revenue NUMERIC,
	transaction_revenue NUMERIC,
	transaction_id VARCHAR,
	page_title VARCHAR,
	search_keyword VARCHAR,
	page_path_level1 VARCHAR,
	ecommerce_action_type INT,
	ecommerce_action_step INT,
	ecommerce_action_option VARCHAR
	);
	
COPY all_sessions(
	full_visitor_id,
	channel_grouping,
	visit_time,
	country,
	city,
	total_transaction_revenue,
	transactions,
	time_on_site,
	page_views,
	session_quality_dim,
	visit_date,
	visit_id,
	visit_type,
	product_refund_amount,
	product_quantity,
	product_price,
	product_revenue,
	product_sku,
	v2product_name,
	v2product_category,
	product_variant,
	currency_code,
	item_quantity,
	item_revenue,
	transaction_revenue,
	transaction_id,
	page_title,
	search_keyword,
	page_path_level1,
	ecommerce_action_type,
	ecommerce_action_step,
	ecommerce_action_option)
FROM 'D:\LHL\projects\data\all_sessions.csv'
DELIMITER ','
CSV HEADER;
```

### Understanding the data
First, I explored and inspected the data in each table to become familiar with its contents. I examined the columns in each table to ensure that the data types specified during loading were correct or to determine if they needed adjustment. I also assessed how the data could be better formatted, identified unnecessary data that could be removed, and considered how the tables and columns could relate to each other.

### Data cleaning
I decided to create views of the tables for my analysis instead of altering the original tables. Views provide a way to present transformed or filtered data without modifying the underlying data. This allows for flexible querying and analysis while maintaining the integrity and consistency of the source data. 

### Analizing the data and answering questions 
I have analyzed the data to answer the asked questions. The results can be found in starting_with_data.md and starting_with_questions.md files.

### QA process


## Results
The data we worked with seemed to be from an online store. Here's what I did:

To find out which cities and countries had the highest transaction revenues (question 1), I used functions that add up values. I used COALESCE to handle cases where data was missing.

For questions 3 and 4, I used CTEs with window functions. This helped me count how many of each product category were sold in different cities and countries.

Question 5 involved calculating total revenue by country and figuring out what percentage of the total revenue each country contributed.


## Challenges 
- Due to time constraints, I was unable to fully explore, analyze, and clean the data.
- Unfamiliarity with the context of the data made it challenging to analyze effectively.
- Working with unorganized data containing large amount of null values. 

## Future Goals
- Investigating the numerous null values to determine whether to impute them, leave them as is, or analyze what they might indicate.
- Exploring other tables in more depth, as I primarily used the all_sessions table for my project.
- Cleaning the data more thoroughly.
