## What issues will you address by cleaning the data?

## 1. Missing values

Firstly, I have inspected each table for missing values with the query below.

```SQL
SELECT COUNT(*)
FROM sales_report
WHERE product_sku IS NULL; 
```
Based on the column content, I've decided that columns containing only null values (e.g., user_id) or columns with consistent single values throughout (e.g., social_engagement_type) will not be included in the analysis. Consequently, these columns have been dropped or, in my case, excluded from the cleaned table views.

I dropped the following columns: 
- from all_sessions table: search_keyword, item_revenue, item_quantity, product_refund_amount
- from analytics: user_id and social_engagement_type

However, there is still a significant number of null values across the tables. I have chosen not to remove them at this stage because they may provide valuable insights, depending on the specific questions being addressed in the analysis. Understanding the context and potential significance of these null values is important before deciding on their removal. 

## 2. Data type validation and conversion

I believe that the data types assigned during import (VARCHAR, INT, DATE, etc.) generally match the actual data stored. However, I acknowledge that it could be beneficial to refine these assignments by specifying exact character lengths for VARCHAR columns, rather than defaulting to VARCHAR(255), to optimize storage and ensure consistency across the database.

## 3. Inconsistent Formatting

- removing white spaces:

```SQL
select TRIM(product_name) from sales_report
```

- changing units:

I adjusted unreasonably high prices in the unit_price column of the analytics table, and in product_revenue, product_price, transaction_revenue, and total_transaction_revenue columns of the all_sessions table by dividing them by 1,000,000.

```SQL
unit_price / 1000000 AS unit_price
```
-  for the city column in all_sessions table I changed '(not set)' and 'not available in demodataset' values to NULL for cconsistency

```SQL
CASE WHEN city = '(not set)' OR city = 'not available in demo dataset' THEN NULL
ELSE city
END AS city
```

- aligning the same values but named differently to one version:
```SQL
CASE 
WHEN v2product_category = '(not set)' OR v2product_category = '${escCatTitle}' THEN NULL
WHEN v2product_category = 'Nest-USA' THEN 'Home/Nest/Nest-USA'
WHEN v2product_category = 'YouTube' THEN 'Home/Brands/YouTube'
WHEN v2product_category = 'Apparel' THEN 'Home/Apparel'
WHEN v2product_category = 'Bags' THEN 'Home/Home/Bags'
WHEN v2product_category LIKE '%Bottles%' THEN 'Home/Drinkware/Water Bottles and Tumblers'
WHEN v2product_category = 'Drinkware' THEN 'Home/Drinkware'
WHEN v2product_category = 'Electronics' THEN 'Home/Electronics'
WHEN v2product_category = 'Headgear' THEN 'Home/Apparel/Headgear'
WHEN v2product_category = 'Housewares' THEN 'Home/Accessories/Housewares'
WHEN v2product_category = 'Office' THEN 'Home/Office'
WHEN v2product_category = 'Waze' THEN 'Home/Shop by Brand/Waze'
WHEN v2product_category LIKE '%Wearables/Men''s T-Shirts%' THEN 'Home/Apparel/Men''s/Men''s-T-Shirts'
ELSE CAST (TRIM(TRAILING '/' FROM TRIM(v2product_category)) AS VARCHAR)
```
## 3. Duplicates

The sample code below can be used to examine and address duplicate values. I have identified duplicate rows in the analytics table. Based on this analysis and the close examination of the duplicates, a decision may be made to delete these repeated rows.

```SQL
SELECT visit_id, COUNT(*)
FROM all_sessions
GROUP BY visit_id
HAVING COUNT(*) > 1;
```
I found a duplicated transaction_revenue column in the all_sessions table, which only contained 4 values identical to those in total_transactions_revenue. Therefore, I decided to remove this redundant column.

```SQL
SELECT
    total_transaction_revenue, 
    transaction_revenue
FROM
    all_sessions
WHERE   
    transaction_revenue IS NNOT NULL
```

## 4. Cleaned table views:

```SQL
-- Creating all_sessions view:
CREATE OR REPLACE VIEW all_sessions_clean AS 
SELECT
    full_visitor_id,
    channel_grouping,
    COALESCE(NULLIF(country, '(not set)'), NULL) AS country,
    COALESCE(NULLIF(city, '(not set)'), NULLIF(city, 'not available in demo dataset'), NULL) AS city,
    total_transaction_revenue / 1000000 AS total_transaction_revenue,
    transactions,
    time_on_site,
    page_views,
    session_quality_dim,
    visit_date,
    visit_id,
    visit_type,
    product_quantity,
    product_price / 1000000 AS product_price,
    product_revenue / 1000000 AS product_revenue,
    product_sku,
    TRIM(v2product_name) AS v2product_name,
    CASE
        WHEN v2product_category = '(not set)' OR v2product_category = '${escCatTitle}' THEN NULL
        WHEN v2product_category = 'Nest-USA' THEN 'Home/Nest/Nest-USA'
        WHEN v2product_category = 'YouTube' THEN 'Home/Brands/YouTube'
        WHEN v2product_category = 'Apparel' THEN 'Home/Apparel'
        WHEN v2product_category = 'Bags' THEN 'Home/Home/Bags'
        WHEN v2product_category LIKE '%Bottles%' THEN 'Home/Drinkware/Water Bottles and Tumblers'
        WHEN v2product_category = 'Drinkware' THEN 'Home/Drinkware'
        WHEN v2product_category = 'Electronics' THEN 'Home/Electronics'
        WHEN v2product_category = 'Headgear' THEN 'Home/Apparel/Headgear'
        WHEN v2product_category = 'Housewares' THEN 'Home/Accessories/Housewares'
        WHEN v2product_category = 'Office' THEN 'Home/Office'
        WHEN v2product_category = 'Waze' THEN 'Home/Shop by Brand/Waze'
        WHEN v2product_category LIKE '%Wearables/Men''s T-Shirts%' THEN 'Home/Apparel/Men''s/Men''s-T-Shirts'
        ELSE CAST(TRIM(TRAILING '/' FROM TRIM(v2product_category)) AS VARCHAR)
    END AS v2product_category,
    product_variant,
    currency_code,
    transaction_id,
    page_title,
    page_path_level1,
    ecommerce_action_type,
    ecommerce_action_step,
    ecommerce_action_option
FROM all_sessions;

-- Creating products view:
CREATE OR REPLACE VIEW products_clean AS
SELECT	sku,
		TRIM(product_name) AS product_name,
		ordered_quantity, 
		stock_level,
		restocking_lead_time,
		sentiment_score,
		sentiment_magnitude 
FROM products;

-- Creating analytics view:
CREATE OR REPLACE VIEW analytics_clean AS 
SELECT	visit_number,
		visit_id,
		visit_start_time,
		visit_date,
		full_visitor_id,
		channel_grouping,
		units_sold,
		page_views,
		time_on_site,
		bounces,
		revenue,
		unit_price / 1000000 AS unit_price
FROM   	analytics;
```

