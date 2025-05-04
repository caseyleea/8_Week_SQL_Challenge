# Case Study #1 - Danny's Diner
<img src="https://8weeksqlchallenge.com/images/case-study-designs/1.png" width="600">

This case study was sourced from Data with Danny's [8 Week SQL Challenge](https://8weeksqlchallenge.com/).<br>
For more details about the Danny's Diner case study, such as background and datasets used, click [here](https://8weeksqlchallenge.com/case-study-1/).
___
## Problem Statement
Danny wants to answer questions about his customers, such as their visiting patterns, how much they spend, and their favorite menu items. Using the restaurant's sales, menu, and members data, he wants to use these insights to determine whether he should expand the current customer loyalty program.
___
## Entity Relationship Diagram
<img src="https://user-images.githubusercontent.com/81607668/127271130-dca9aedd-4ca9-4ed8-b6ec-1e1920dca4a8.png" width="600">

___
## Case Study Questions and Solutions

**1. What is the total amount each customer spent at the restaurant?**
```sql
SELECT
  sa.customer_id,
  SUM(me.price) AS total_spent
FROM sales sa
JOIN menu me
  ON sa.product_id = me.product_id
GROUP BY sa.customer_id
ORDER BY total_spent DESC;
```
*Output:*
| customer_id  | total_spent |
| ------------- | ------------- |
| A  | 76 |
| B | 74  |
| C | 36 |

*Answer:*
___
**2. How many days has each customer visited the restaurant?**
```sql
SELECT
	customer_id,
    COUNT(DISTINCT order_date) AS days_visited
FROM dannys_diner.sales
GROUP BY customer_id
ORDER BY days_visited DESC; 
```
*Output:*
| customer_id  | days_visited |
| ------------- | ------------- |
| B  | 6 |
| A | 4  |
| C | 2 |

*Answer:*
___
**3. What was the first item from the menu purchased by each customer?**
```sql
WITH order_rank AS (
SELECT
  sa.customer_id,
  me.product_name,
  sa.order_date,
  DENSE_RANK() OVER (PARTITION BY sa.customer_id ORDER BY sa.order_date)
AS order_number
FROM dannys_diner.sales sa
JOIN dannys_diner.menu me
  ON sa.product_id = me.product_id)

SELECT
  customer_id, 
  product_name,
  COUNT(product_name) AS purchased
FROM order_rank
WHERE order_number = 1
GROUP BY customer_id, product_name; 
```
*Output:*
| customer_id  | product_name |purchased |
| ------------- | ------------- |------------- |
| A  | curry |1 |
| A | sushi  |1 |
| B | curry |1 |
| C | ramen |2 |

*Answer:*
___
**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**
```sql
SELECT
	me.product_name,
	COUNT(sa.product_id) as times_purchased
FROM dannys_diner.sales sa
LEFT JOIN dannys_diner.menu me
  ON sa.product_id = me.product_id
GROUP BY me.product_name
ORDER BY times_purchased DESC
LIMIT 1;
```
*Output:*
| product_name  | times_purchased |
| ------------- | ------------- |
| ramen  | 8 |

*Answer:*
