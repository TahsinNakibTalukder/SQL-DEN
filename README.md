# SQL-DEN
SQL Real world problem solutions!

## Customer Visit Analysis - SQL Query
This repository contains SQL scripts used to analyze customer visits by tracking first-time and repeat visits using MariaDB. The main objective of the query is to flag each customer's first visit and subsequent repeat visits, and then aggregate the total number of new and repeat customers per order date.

## Query Breakdown
Common Table Expressions (CTEs)

# first_visit CTE:

This portion of the query selects the customer_id and the minimum order_date (i.e., the first visit date) from the customer_orders table. It groups the data by customer_id to ensure each customer’s first visit is captured.
sql

WITH first_visit AS (
    SELECT customer_id, MIN(order_date) AS first_visit_date
    FROM customer_orders
    GROUP BY customer_id
)
# visit_flag CTE:

This CTE uses the previous result and joins the customer_orders table with the first_visit CTE on customer_id. Two flags are generated:
first_visit_flag: A flag indicating whether the order_date is the same as the first_visit_date (1 if true, otherwise 0).
repeat_visit_flag: A flag indicating whether the order_date is different from the first_visit_date (1 if true, otherwise 0).
sql

, visit_flag AS (
    SELECT 
        co.*, 
        fv.first_visit_date,
        CASE WHEN co.order_date = fv.first_visit_date THEN 1 ELSE 0 END AS first_visit_flag,
        CASE WHEN co.order_date != fv.first_visit_date THEN 1 ELSE 0 END AS repeat_visit_flag
    FROM customer_orders co 
    INNER JOIN first_visit fv ON co.customer_id = fv.customer_id
    ORDER BY order_id
)
# Aggregation of New and Repeat Customers
In the final SELECT query, the script sums up the first_visit_flag and repeat_visit_flag for each order_date to compute the number of new and repeat customers on that date.
sql

SELECT 
    order_date, 
    SUM(first_visit_flag) AS no_of_new_customers, 
    SUM(repeat_visit_flag) AS no_of_repeat_customers
FROM visit_flag
GROUP BY order_date;
Usage

To use this query, you need a table customer_orders that contains at least the following columns:

customer_id (to uniquely identify each customer)
order_date (date when the order was placed)
order_id (unique identifier for each order)
The query outputs two key metrics for each order date:

no_of_new_customers: The count of first-time customers on that date.
no_of_repeat_customers: The count of repeat customers on that date.

## Requirements

This script is compatible with MariaDB and similar relational databases.
Ensure your database contains appropriate indices on customer_id and order_date for optimized performance.
