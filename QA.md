## What are your risk areas? Identify and describe them.
- Potential risk areas include tables, columns, or rows containing entirely null, irrelevant, or duplicate data. 
- Column formatting, which may involve incorrect data types or inconsistent formats, leading to data integrity issues and inaccurate analyses


## QA Process:

### Null values:

I identified four null tables in 'all_sessions' table:
- search_keyword, 
- item_revenue, 
- item_quantity, 
- product_refund_amount

In the 'analytics' table, there was one null column:
- user_id

For all of these columns, I executed the following query to check if they are null.

```SQL
SELECT
    search_keyword 
FROM
    all_sessions 
WHERE search_keyword IS NOT NULL
```
After running query for each table I confirmed that these columns were null and decided to remove them from tables.

### Duplicate values

I identyfied a redundant transaction_revenue column in the all_sessions table. Before removing from the table I double checked if this is duplicate by comparing the values in this column within other similar columns in the table.

```SQL
SELECT
	total_transaction_revenue,
	transactions, 
	product_revenue, 
	transaction_revenue
FROM
	all_sessions
WHERE
	product_revenue IS NOT NULL
```
### Single values

I identified a social_engagement_type column that had a single consistent value throughout. This column does not provide any meaningful information, so I confirmed with the query below and decided to drop the column.

```SQL
SELECT 
	social_engagement_type
FROM
	analytics
WHERE
	social_engagement_type = 'Not Socially Engaged'
```


