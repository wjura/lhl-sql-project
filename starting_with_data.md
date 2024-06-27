### Starting with data
 1. Find all duplicate records
```SQL
-- checking for duplicates in all_sessions table; no duplicates
SELECT 
	*, 
	COUNT(*)
FROM 
	all_sessions
GROUP BY 
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
	ecommerce_action_option
HAVING COUNT(*) > 1;

--checking for duplicates in products table; no duplicates
SELECT 
	*, 
	COUNT(*)
FROM 
	products
GROUP BY
	sku,
	product_name, 
	ordered_quantity, 
	stock_level,
	restocking_lead_time,
	sentiment_score,
	sentiment_magnitude
HAVING COUNT(*) > 1;

--checking for duplicates in sales_report table; no duplicates
SELECT 
	*, 
	COUNT(*)
FROM 
	sales_report
GROUP BY
	product_sku,
	total_ordered, 
	product_name, 
	stock_level,
	restocking_lead_time,
	sentiment_score,
	sentiment_magnitude,
	ratio
HAVING COUNT(*) > 1;

--checking for duplicates in sales_by_sku table; no duplicates
SELECT 
	*, 
	COUNT(*)
FROM 
	sales_by_sku
GROUP BY
	product_sku,
	total_ordered
HAVING COUNT(*) > 1;

--checking for duplicates in analytics table; 870577 duplicates
SELECT 
	*, 
	COUNT(*)
FROM 
	analytics
GROUP BY
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
	unit_price
HAVING COUNT(*) > 1;
```

2. Find the total number of unique visitors (`fullVisitorID`)

```SQL
SELECT 
	COUNT(full_visitor_id)
FROM 
	all_sessions;				
    
--total number of visitors: 15134
	
SELECT 
	COUNT(DISTINCT full_visitor_id)
FROM 
	all_sessions; 				
    
-- total number of unique visitors: 14223; might be that some people vistited more than once
```

3. Find the total number of unique visitors by referring sites

```SQL
SELECT
    channel_grouping,
    COUNT(DISTINCT full_visitor_id) AS unique_visitors
FROM
    all_sessions
WHERE
    channel_grouping = 'Referral'
GROUP BY
    channel_grouping;			

--unique_visitors: 2419
```

4. Find each unique product viewed by each visitor
```SQL
SELECT
    full_visitor_id,
    v2product_name,
    COUNT(*) AS view_count
FROM
    all_sessions
WHERE
    full_visitor_id IS NOT NULL
    AND v2product_name IS NOT NULL
GROUP BY
    full_visitor_id, 
	v2product_name
ORDER BY
    full_visitor_id, 
	view_count DESC;	
    
-- it appears, that each unique product was viewed only once by each visitor
```

 5. Compute the percentage of visitors to the site that actually makes a purchase

```SQL
SELECT
    ROUND(COUNT(DISTINCT CASE WHEN transactions > 0 THEN full_visitor_id END) * 100.0 / COUNT(DISTINCT full_visitor_id),2) AS visitors_with_purchase_percent
FROM
    all_sessions
WHERE
    full_visitor_id IS NOT NULL;

-- 0.56% visitors made a purchase
```
