# Cleaning-and-Analyzing-Data-with-SQL
Answering business questions by analyzing data with SQL 


CREATE DATABASE autosales;

USE autosales;
-----------------------------------------------

## SOME DATA CLEANING 

-- updating column name
```SQL
ALTER TABLE products
RENAME COLUMN ï»¿order_number TO order_number;
```

-- convert text to date type
ALTER TABLE products
MODIFY COLUMN order_date DATE ;

DESC products;
------------------------------------------------------


## 20 SQL QUESTIONS AND ANSWERS:


### CUSTOMER ANALYTICS SECTION
#### 1. What is the total number of customers by region

SELECT 
	CASE
		WHEN country IN ('USA', 'Canada') THEN 'North America'
        WHEN country IN ('France', 'Norway', 'Austria', 'Finland', 'UK', 'Spain', 'Sweden', 
						'Italy', 'Denmark', 'Belgium', 'Germany', 'Switzerland', 'Ireland') THEN 'Europe'
        WHEN country IN ('Philippines', 'Japan', 'Singapore') THEN 'Asia' 
        WHEN country IN ('Australia') THEN 'Oceania' 
		ELSE 'Other' 
	END AS region,
    -- country, 
    COUNT(DISTINCT customer_name) AS total_customers
FROM
    customers
GROUP BY region
ORDER BY total_customers DESC;

#### 2. What are the top 10 cities with the most customers?

SELECT 
    city, COUNT(DISTINCT customer_name) AS customer_in_city
FROM
    customers
GROUP BY city
ORDER BY customer_in_city DESC
LIMIT 10;

#### 3. What is the distribution of orders based on deal size (Small, Medium, Large)?

SELECT 
    COUNT(DISTINCT order_number) AS count_of_orders, dealsize
FROM
    customers
GROUP BY dealsize
ORDER BY count_of_orders DESC;

#### 4. What is the total number of customers per country?

SELECT country, COUNT(DISTINCT customer_name) AS number_of_customers
FROM customers
GROUP BY country
ORDER BY number_of_customers DESC;

#### 5. Which country has the highest revenue generation based on sales?

SELECT 
	c.country, ROUND(SUM(p.sales), 2) AS total_sales
FROM
    customers c
        JOIN
    products p ON c.order_number = p.order_number
GROUP BY country
ORDER BY total_sales DESC
LIMIT 1;


### ORDER AND SALES ANALYSIS

#### 6. What is the overall total sales?

SELECT
	ROUND(SUM(sales),2) AS total_sales
FROM 
	products;

#### 7. What is the overall total orders?
SELECT
	COUNT(DISTINCT order_number) AS total_sales
    -- ROUND(SUM(quantity_ordered)) AS total_quantity_ordered
FROM 
	products;

#### 8. What is the overall total quantity ordered?
SELECT
    SUM(quantity_ordered) AS total_quantity_ordered
FROM 
	products;

#### 9. What is the total numder of customers of the company?
SELECT
	COUNT(DISTINCT customer_name) AS total_customer
FROM 
	 customers;

#### 10. What is the total sales amount per customer?

SELECT 
    c.customer_name, ROUND(SUM(p.sales), 2) AS total_sales
FROM
    customers c
        JOIN
    products p ON c.order_number = p.order_number
GROUP BY customer_name
ORDER BY total_sales DESC;

#### 11. What is the average order value (AOV) per customer?

SELECT 
	c.customer_name,
    ROUND(SUM(p.sales),2) AS total_sales,
    COUNT(DISTINCT p.order_number) AS total_orders,
	ROUND(SUM(p.sales) / COUNT(DISTINCT p.order_number)) AS average_ordered_value
FROM customers c 
	JOIN
    products p ON c.order_number = p.order_number
GROUP BY c.customer_name
ORDER BY total_orders DESC;

#### 12. What number of orders are placed in each country?

SELECT c.country,
	COUNT(DISTINCT p.order_number) AS number_of_orders
FROM customers c 
	JOIN
    products p ON c.order_number = p.order_number
GROUP BY country
ORDER BY number_of_orders DESC;

#### 13. Which customers contribute the most to total sales?

SELECT 
	customer_name,
	ROUND(SUM(p.sales),2) AS total_sales
FROM customers c 
	JOIN
    products p ON c.order_number = p.order_number
GROUP BY customer_name
ORDER BY total_sales DESC
LIMIT 10;

#### 14. What is the trend of sales over time (monthly)?

SELECT 
	YEAR(order_date) AS year,
    DATE_FORMAT(order_date, '%M %Y') AS month_name,
    ROUND(SUM(sales),2) AS total_sales
FROM
    products
GROUP BY YEAR(order_date) , MONTH(order_date), month_name
ORDER BY YEAR(order_date) , MONTH(order_date);


### PRODUCT INSIGHT

#### 15. What are the top 5 most sold products based on quantity?

SELECT 
    product_code, 
    SUM(quantity_ordered) AS total_quantity
FROM
    products
GROUP BY product_code
ORDER BY total_quantity DESC
LIMIT 5;
    
#### 16. What is the total sales amount or revenue per each product line?

SELECT 
    product_line, 
    ROUND(SUM(sales),2) AS total_sales
FROM
    products
GROUP BY product_line
ORDER BY total_sales DESC;

#### 17. What is the average sales per product line?

SELECT
	product_line,
	ROUND(AVG(sales),2) productline_average_sales
FROM products
GROUP BY product_line
ORDER BY productline_average_sales DESC;

#### 18. What is the average sales per product line yearly?

SELECT 
	year(order_date) AS year_ordered,
	product_line,
	ROUND(AVG(sales),2) productline_average_sales
FROM products
GROUP BY year_ordered, product_line
ORDER BY year_ordered, productline_average_sales DESC;

#### 19. How does the manufacturer's suggested retail price (MSRP) compare to actual sales prices?✔

SELECT 
	product_code,
    man_suggested_retail_price,
    ROUND(AVG(price_each), 2) AS avg_price_per_item,
    ROUND((man_suggested_retail_price - AVG(price_each)), 2) AS price_difference,
    CONCAT(ROUND(((man_suggested_retail_price - AVG(price_each)) / man_suggested_retail_price) * 100), '%') AS percentage_difference
FROM
    products
GROUP BY product_code, man_suggested_retail_price;

#### 20. What is the average quantity ordered for each product line?✔

SELECT
	product_line,
    ROUND(AVG(quantity_ordered),2) AS average_quatity_ordered
FROM products
GROUP BY product_line
ORDER BY average_quatity_ordered DESC;


/*SELECT
	product_line,
    ROUND(SUM(quantity_ordered),2) AS sum_quatity_ordered,
    ROUND(count(quantity_ordered),2) AS numner_quatity_ordered
FROM products
GROUP BY product_line
ORDER BY sum_quatity_ordered DESC;*/


### ORDER TRENDS

#### 21. How does the frequency of orders change over time?

SELECT
	YEAR(order_date) AS order_year,
    COUNT(DISTINCT order_number) AS total_unique_orders
FROM products
GROUP BY YEAR(order_date)
ORDER BY total_unique_orders DESC;

#### 22. Which order status are most common?

SELECT
	status,
    COUNT(status) AS status_count
FROM products
GROUP BY status
ORDER BY status_count DESC;

#### 23. What is the percentage contribution of each product line to total sales?

SELECT
	product_line,
    ROUND(SUM(sales),2) AS total_sales,
	CONCAT(ROUND((SUM(sales) / (SELECT SUM(sales) FROM products)) * 100, 2), '%') AS percentage_contribution
FROM products
GROUP BY product_line
ORDER BY total_sales DESC;


-- OR using CTE 


WITH total_sales AS (
	SELECT
		ROUND(SUM(sales),2) AS overall_total_sales
	FROM products
),
product_line_sales AS (
	SELECT
		product_line,
        ROUND(SUM(sales),2) AS total_sales
	FROM products
    GROUP BY product_line
)
SELECT 
	p.product_line,
    p.total_sales,
    CONCAT(ROUND((p.total_sales / t.overall_total_sales) * 100, 2), '%') AS percentage_contribution
FROM 
	total_sales t
    CROSS JOIN
    product_line_sales p 
ORDER BY p.total_sales DESC;

#### 24. What is the total revenue per country, and how does it vary by region?

SELECT
	CASE
		WHEN country IN ('USA', 'Canada') THEN 'North America'
        WHEN country IN ('France', 'Norway', 'Austria', 'Finland', 'UK', 'Spain', 'Sweden', 
						'Italy', 'Denmark', 'Belgium', 'Germany', 'Switzerland', 'Ireland') THEN 'Europe'
        WHEN country IN ('Philippines', 'Japan', 'Singapore') THEN 'Asia' 
        WHEN country IN ('Australia') THEN 'Oceania' 
		ELSE 'Other' 
	END AS region,
    country,
    ROUND(SUM(sales),2) AS total_revenue
FROM customers c 
	JOIN
    products p ON c.order_number = p.order_number
GROUP BY region, country
ORDER BY region, total_revenue DESC;

/* 
North America - USA, Canada
Europe - France, Norway, Austria, Finland, UK, Spain, Sweden, Italy, Denmark, Belgium, Germany, Switzerland, Ireland
Asia - Philipines, Japan, Singapore
Oceania - Australia

 */



