#  🍛 Case Study: Danny Dinner
CREDIT: Danny Ma.
You can view his challenge here: https://8weeksqlchallenge.com/case-study-1/

# Overview:
Use the data to answer a few simple questions about Danny's customers
  1. Visiting patterns
  2. Amount customers have spent
  3. Top favorite products

### 1. What is the total amount each customer spent at the restaurant?
#### Steps:
- Use **SUM** and **GROUP BY** to find out ```total_sales``` contributed by each customer.
- Use **JOIN** to merge ```sales``` and ```menu``` tables as ```customer_id``` and ```price``` are from both tables.
````sql
SELECT
    customer_id,
    SUm(price) as total_sales
FROM sales 
LEFT JOIN menu
    ON sales.product_id = menu.product_id
GROUP BY customer_id
ORDER BY total_sales DESC;
````
| customer_id | total_sales |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

### 2. How many days has each customer visited the restaurant?
#### Steps:
- Use **DISTINCT** and wrap with **COUNT** to find out the ```visit_count``` for each customer.
- If we do not use **DISTINCT** on ```order_date```, the number of days may be repeated. For example, if Customer A visited the restaurant twice on '2021–01–07', then number of days is counted as 2 days instead of 1 day.
````sql
SELECT
    sales.customer_id,
    COUNT (distinct order_date) as total_date
FROM sales
GROUP BY sales.customer_id
ORDER BY total_date DESC;
````
| customer_id | visit_count |
| ----------- | ----------- |
| A           | 4          |
| B           | 6          |
| C           | 2          |
### 3. What was the first item from the menu purchased by each customer?
#### Steps:
- Use CTE and RANK() to rank order by each customers
- Use SELECT DISTINCT to find the first item purchased
````sql
WITH ranked_order AS (
SELECT
    customer_id,
    Rank() OVER(PARTITION BY customer_id ORDER BY order_date ASC) AS order_rank,
    product_name
FROM sales
JOIN menu
 ON sales.product_id = menu.product_id
)
SELECT DISTINCT
    customer_id,
    product_name
FROM ranked_order
WHERE order_rank = 1;

--- 
Select TOP 1
    product_name,
    COunt(*) as count_sales
From sales
JOIN menu
    On sales.product_id = menu.product_id
GROUP BY product_name
ORDER BY count_sales DESC;
````
| customer_id | product_name | 
| ----------- | ----------- |
| A           | curry        | 
| A           | sushi        | 
| B           | curry        | 
| C           | ramen        |
### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
#### Steps:
- **COUNT** number of ```product_id``` and **ORDER BY** ```most_purchased``` by descending order. 
- Then, use **TOP 1** to filter highest number of purchased item.

````sql
Select TOP 1
    product_name,
    COunt(*) as count_sales
From sales
JOIN menu
    On sales.product_id = menu.product_id
GROUP BY product_name
ORDER BY count_sales DESC;
````
| most_purchased | product_name | 
| ----------- | ----------- |
| 8       | ramen |
### 5. Which item was the most popular for each customer?
#### Steps:
- Create a ```fav_item_cte``` and use **DENSE_RANK** to ```rank``` the ```order_count``` for each product by descending order for each customer.
- Generate results where product ```rank = 1``` only as the most popular product for each customer.
````sql
WITH fav_item_cte AS (
  SELECT s.customer_id, 
        product_name, 
        COUNT(m.product_id) AS order_count, 
        Rank() OVER(PARTITION BY customer_id ORDER BY COUNT(m.product_id) DESC) AS rank
  FROM sales AS s
  JOIN menu AS m
      ON s.product_id = m.product_id
  GROUP BY s.customer_id, product_name
)
SELECT customer_id,
        product_name,
        order_count
FROM fav_item_cte
WHERE rank = 1;
````
| customer_id | product_name | order_count |
| ----------- | ---------- |------------  |
| A           | ramen        |  3   |
| B           | sushi        |  2   |
| B           | curry        |  2   |
| B           | ramen        |  2   |
| C           | ramen        |  3   |
