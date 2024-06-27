# Starting with questions



Answer the following questions and provide the SQL queries used to find the answer.

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**



```SQL
SELECT 
	city, 
	SUM(total_transaction_revenue)
FROM 
	all_sessions_clean
WHERE 
	total_transaction_revenue IS NOT NULL
	AND city IS NOT NULL
GROUP BY 
	city
ORDER BY 
	SUM(total_transaction_revenue) DESC
LIMIT 5;

-- Top 5 cities :San Francisco, Sunnyvale, Atlanta, Palo Alto, Tel Aviv-Yafo
```

```SQL
SELECT 
	DISTINCT country, 
	SUM(total_transaction_revenue)
FROM 
	all_sessions_clean
WHERE 
	total_transaction_revenue IS NOT NULL
GROUP BY 
	country
ORDER BY 
	SUM(total_transaction_revenue) DESC;

-- Top 5 countries: United States, Israel, Australia, Canada, Switzerland
```

**Question 2: What is the average number of products ordered from visitors in each city and country?**

It is somewhat ambiguous which columns to include in the further analysis to answer this and following questions, as there are similar columns in different tables: product_quantity in all_sessions, total_ordered in sales_by_sku, total_ordered in sales_report, and ordered_quantity in products. 

After investigating these columns, I have decided to base the analysis on records where transactions are reflected in the transactions column of the all_sessions table, as these align with the total_transaction_revenue column. Additionally, I will consider product_quantity, which is supposed to indicate the ordered quantity, although the majority of values are NULL. 

I considered imputing missing values by dividing total_transaction_revenue by product_price, but discrepancies arose, possibly due to taxes or discounts being included in one amount but not the other. Uncertain about the exact nature of these discrepancies, I chose to keep the data unchanged. I recognize that this decision affects the completeness of my analysis due to the missing values, but I will proceed with the available data.

```SQL
-- Average by city and country:

SELECT  
	city,
	country,
	AVG(COALESCE(product_quantity,0)) avg_ordered
FROM 
	all_sessions_clean
WHERE 
	transactions > 0
	AND city IS NOT NULL
GROUP BY 
	city,
    country
ORDER BY 
	avg_ordered DESC;

-- Query returns 20 records	with half of them being 0. The highest average is for Atlanta. This might suggest that most transactions did not record quantities ordered.
	
-- Average by country:

SELECT  
	country,
	AVG(COALESCE(product_quantity,0)) avg_ordered
FROM 
	all_sessions_clean
WHERE 
	transactions > 0
GROUP BY 
    country
ORDER BY 
	avg_ordered DESC;

-- Query returns 5 records, but only the top one, United States, has an average above 0. It seems there is a discrepancy in the data for Australia, Canada, Israel and Switzerland, where transactions were registered but no quantity ordered was recorded. Further investigation is needed to understand the reasons behind these discrepancies and to ensure data accuracy and consistency.
```

**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**

```SQL
-- Category by city:

WITH ranked_categories AS (
	SELECT
		city,
		country,
		v2product_category,
		COUNT(*) AS order_count,
		RANK() OVER (PARTITION BY city, country ORDER BY COUNT(*) DESC) AS rank
	FROM 
		all_sessions_clean
	WHERE 
        transactions > 0
	GROUP BY 
		city,
		country,
		v2product_category
)
SELECT 
	city,
	country,
	v2product_category,
	order_count
FROM 
	ranked_categories
WHERE 
	rank = 1
	AND city IS NOT NULL
	AND country IS NOT NULL
	AND v2product_category IS NOT NULL
GROUP BY
	city,
	country,
	v2product_category,
	order_count
ORDER BY 
	order_count DESC,
	country;

-- This query returns the 9 records across which the most common category is Home/Nest/Nest-USA.
```
```SQL
-- Category by country:

WITH ranked_categories AS (
	SELECT
		country,
		v2product_category,
		SUM(product_quantity) AS total_ordered,
		RANK() OVER (	PARTITION BY country
						ORDER BY SUM(product_quantity) DESC) AS rank
	FROM 
		all_sessions_clean
	WHERE
		transactions > 0
	GROUP BY 
		country,
		v2product_category
	HAVING 
		SUM(product_quantity) IS NOT NULL
)
SELECT 
	country,
	v2product_category,
	total_ordered
FROM 
	ranked_categories
WHERE 
	rank = 1
	AND v2product_category IS NOT NULL
GROUP BY
	country,
	v2product_category,
	total_ordered
ORDER BY 
	total_ordered DESC;

-- Query returns only one record, as the only country with recorded quantity sold is United States. The most popular category is Home/Office/Notebooks & Journals.
```

**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**

```SQL
-- Product by city:

WITH product_sales AS (
	SELECT
		city,
		v2product_name,
		SUM(product_quantity) AS order_total,
		RANK() OVER (	PARTITION BY city 
						ORDER BY SUM(product_quantity)DESC) AS rank
	FROM 
		all_sessions_clean
	WHERE 
		transactions > 0
		AND city IS NOT NULL
	GROUP BY 
		city,
		v2product_name
	HAVING SUM(product_quantity) IS NOT NULL
)
SELECT 
	city,
	v2product_name,
	order_total
FROM 
	product_sales
WHERE 
	rank = 1
	AND city IS NOT NULL
GROUP BY
	city,
	v2product_name,
	order_total
ORDER BY 
	order_total

-- The most popular product is Learning Thermostat.

-- Product by country:
WITH product_sales AS (
	SELECT
		country,
		v2product_name,
		SUM(product_quantity) AS order_total,
		RANK() OVER (	PARTITION BY country 
						ORDER BY SUM(product_quantity)DESC) AS rank
	FROM 
		all_sessions_clean
	WHERE 
		transactions > 0
	GROUP BY 
		country,
		v2product_name
	HAVING SUM(product_quantity) IS NOT NULL

)
SELECT 
	country,
	v2product_name,
	order_total
FROM 
	product_sales
WHERE 
	rank = 1
GROUP BY
	country,
	v2product_name,
	order_total
ORDER BY 
	order_total

-- Query returns only one record (due to majority missing quantities) showing that the most ordered product in United States is Leatherette Journal.
```

**Question 5: Can we summarize the impact of revenue generated from each city/country?**
```SQL
WITH total_rev_country AS (
    SELECT
        country,
        SUM(total_transaction_revenue) as total_revenue
    FROM 
        all_sessions_clean
    GROUP BY
        country
    HAVING
        SUM(total_transaction_revenue) IS NOT NULL
)
SELECT
    country, 
    ROUND(total_revenue *100 / (SELECT SUM(total_revenue) FROM total_rev_country), 4) AS proportion_revenue
FROM 
    total_rev_country
ORDER BY 
    proportion_revenue DESC;

-- Most of the revenue comes from United States - it is 92%.

```







