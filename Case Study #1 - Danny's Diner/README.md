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
* Customer A spent $76.
* Customer B spent $74.
* Customer C spent $36.
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
* Customer A visited the restaurant 4 times.
* Customer B visited the restaurant 6 times.
* Customer C visited the restaurant 2 times.
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
* Customer A purchased curry and sushi.
* Customer B purchased curry.
* Customer C purchased ramen.
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
* Ramen is the most sold item on the menu. It was purchased 8 times.
___
**5. Which item was the most popular for each customer?**
```sql
WITH dish_rank AS (
SELECT
  sa.customer_id,
  me.product_name,
  DENSE_RANK () OVER(PARTITION BY sa.customer_id ORDER BY COUNT(sa.product_id)) AS ranking
FROM dannys_diner.sales sa
JOIN dannys_diner.menu me
  ON sa.product_id = me.product_id
GROUP BY sa.customer_id, me.product_name
ORDER BY sa.customer_id)

SELECT
  customer_id,
  product_name
FROM dish_rank
WHERE ranking = 1; 
```
*Output:*
| customer_id  | product_name |
| ------------- | ------------- |
| A | sushi |
| B  | ramen |
| B  | curry |
| B | sushi |
| C | ramen |

*Answer:* CHECK!
* Sushi is the most popular for Customer A.
* Ramen, curry, and sushi are popular for Customer B.
* Ramen is the most popular for Customer C.
___
**6. Which item was purchased first by the customer after they became a member?**
```sql
SELECT
  sa.customer_id, 
  mem.join_date,
  sa.order_date,
  me.product_name,
  DENSE_RANK() OVER (PARTITION BY sa.customer_id ORDER BY sa.order_date) AS rank
FROM dannys_diner.members mem
JOIN dannys_diner.sales sa
  ON sa.customer_id = mem.customer_id
JOIN dannys_diner.menu me
  ON sa.product_id = me.product_id
WHERE sa.order_date > mem.join_date)

SELECT
  customer_id, 
  join_date,
  order_date,
  product_name
FROM member_dish
WHERE rank = 1; 
```
*Output:*
| customer_id  | join_date |order_date |product_name |
| ------------- | ------------- |------------- |------------- |
| A | 2021-01-07T00:00:00.000Z |2021-01-10T00:00:00.000Z |ramen |
| B  | 2021-01-09T00:00:00.000Z |2021-01-11T00:00:00.000Z |sushi |

*Answer:*
* Customer A's first purchase after becoming a member was ramen.
* Customer B's first purchase after becoming a member was sushi.
___
**7. Which item was purchased just before the customer became a member?**
```sql
WITH member_purchase AS (
SELECT
  sa.customer_id, 
  mem.join_date,
  sa.order_date,
  me.product_name,
  DENSE_RANK() OVER (PARTITION BY sa.customer_id ORDER BY sa.order_date DESC) AS rank
FROM dannys_diner.members mem
JOIN dannys_diner.sales sa
  ON sa.customer_id = mem.customer_id
JOIN dannys_diner.menu me
  ON sa.product_id = me.product_id
WHERE sa.order_date < mem.join_date)

SELECT
  customer_id, 
  join_date,
  order_date,
  product_name
FROM member_purchase
WHERE rank = 1;
```
*Output:*
| customer_id  | join_date |order_date |product_name |
| ------------- | ------------- |------------- |------------- |
| A | 2021-01-07T00:00:00.000Z |2021-01-01T00:00:00.000Z |sushi |
| A  | 2021-01-07T00:00:00.000Z |2021-01-01T00:00:00.000Z |curry |
| B  | 2021-01-09T00:00:00.000Z|2021-01-04T00:00:00.000Z |sushi |

*Answer:* CHECK!
* Customer A ordered sushi and curry just before becoming a member.
* Customer B ordered sushi just before becoming a member.
___
**8. What is the total items and amount spent for each member before they became a member?**
```sql
SELECT
  sa.customer_id, 
  mem.join_date,
  COUNT (sa.product_id) AS total_items, 
  SUM(me.price) AS total_spent
FROM dannys_diner.members mem
JOIN dannys_diner.sales sa
  ON sa.customer_id = mem.customer_id
JOIN dannys_diner.menu me
  ON sa.product_id = me.product_id
WHERE sa.order_date < mem.join_date
GROUP BY sa.customer_id, mem.join_date;
```
*Output:*
| customer_id  | join_date |total_items |total_spent |
| ------------- | ------------- |------------- |------------- |
| A | 2021-01-07T00:00:00.000Z |2 |25 |
| B | 2021-01-09T00:00:00.000Z |3 |40|

*Answer:*
* Customer A ordered 2 items and spent $25 before becoming a member.
* Customer B ordered 3 items and spent $40 before becoming a member.
___
**9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**
```sql
WITH purchase_count AS (
SELECT
  sa.customer_id, 
  me.product_name,
  me.price,
  COUNT(sa.product_id) AS purchased
FROM dannys_diner.sales sa
JOIN dannys_diner.menu me
  ON sa.product_id = me.product_id
GROUP BY sa.customer_id, me.product_name, me.price
ORDER BY sa.customer_id, purchased)

SELECT 
  customer_id,
  SUM(points) AS total_points
FROM(
SELECT
  customer_id, 
  product_name,
CASE 
  WHEN product_name = 'sushi' THEN ((purchased*price)*10)*2
  ELSE (purchased*price)*10 END AS points
FROM purchase_count) AS points_calc
GROUP BY customer_id; 
```
*Output:*
| customer_id  | total_points |
| ------------- | ------------- |
| A | 860|
| B | 940|
| C | 360|

*Answer:*
* Customer A has 860 points.
* Customer B has 940 points.
* Customer C has 360 points.
___
**10. n the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**
```sql
WITH purchase_count AS (
SELECT
  sa.customer_id, 
  me.product_name,
  me.price,
  COUNT(sa.product_id) AS purchased
FROM dannys_diner.sales sa
JOIN dannys_diner.menu me
  ON sa.product_id = me.product_id
GROUP BY sa.customer_id, me.product_name, me.price
ORDER BY sa.customer_id, purchased)

SELECT 
  customer_id,
  SUM(points) AS total_points
FROM(
SELECT
  customer_id, 
  product_name,
CASE 
  WHEN product_name = 'sushi' THEN ((purchased*price)*10)*2
  ELSE (purchased*price)*10 END AS points
FROM purchase_count) AS points_calc
GROUP BY customer_id; 
```
*Output:*
| customer_id  | total_points |
| ------------- | ------------- |
| A | 860|
| B | 940|
| C | 360|

*Answer:* CHECK! 
* Customer A has 860 points.
* Customer B has 940 points.
* Customer C has 360 points.
