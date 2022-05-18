# :takeout_box: Case Study #1: Danny's Diner 🍜

## Solution

### 1. What is the total amount each customer spent at the restaurant?

````sql
SELECT customer_id, SUM(price) AmountSpent
FROM sales s JOIN menu m
	 ON s.product_id = m.product_id
GROUP BY customer_id
````

#### Answer:

| customer_id | total_sales |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

***

### 2. How many days has each customer visited the restaurant?

````sql
SELECT customer_id, COUNT(DISTINCT order_date) TotalDays
FROM sales 
GROUP BY customer_id
````

#### Answer:
| customer_id | visit_count |
| ----------- | ----------- |
| A           | 4          |
| B           | 6          |
| C           | 2          |

***

### 3. What was the first item from the menu purchased by each customer?
````sql
SELECT DISTINCT customer_id, order_date, product_name
FROM (
	SELECT customer_id, order_date, product_name,
		  rank() OVER(PARTITION BY customer_id ORDER BY order_date) rank
	FROM sales s JOIN menu m
		 ON s.product_id = m.product_id
	 ) a
WHERE rank = 1
````

#####  You can use a group by instead of the DISTINCT statement

#### Answer:
| customer_id | product_name | 
| ----------- | ----------- |
| A           | curry        | 
| A           | sushi        | 
| B           | curry        | 
| C           | ramen        |

***

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

````sql
SELECT TOP 1 product_name, COUNT(*) NumOfOrders
FROM sales s JOIN menu m
		 ON s.product_id = m.product_id
GROUP BY product_name
ORDER BY NumOfOrders DESC
````

#### Answer:
| most_purchased | product_name | 
| ----------- | ----------- |
| 8       | ramen |

***

### 5. Which item was the most popular for each customer?

````sql
 	WITH CTE_PrCnt AS
		(
		SELECT customer_id, product_name, COUNT(s.product_id) NumOfProducts,
			   DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY COUNT(s.product_id)) rank
		FROM sales s JOIN menu m
			 ON s.product_id = m.product_id
		GROUP BY customer_id, product_name
		)
	SELECT customer_id, product_name, NumOfProducts
	FROM CTE_PrCnt
	WHERE rank = 1
````

#### Answer:
| customer_id | product_name | order_count |
| ----------- | ---------- |------------  |
| A           | ramen        |  3   |
| B           | sushi        |  2   |
| B           | curry        |  2   |
| B           | ramen        |  2   |
| C           | ramen        |  3   |

***

### 6. Which item was purchased first by the customer after they became a member?
	
  ````sql
  WITH member_sales_cte AS 
	(
	   SELECT s.customer_id, m.join_date, s.order_date, s.product_id,
		  DENSE_RANK() OVER(PARTITION BY s.customer_id
		  ORDER BY s.order_date) AS rank
	   FROM sales AS s
	   JOIN members AS m
		  ON s.customer_id = m.customer_id
	   WHERE s.order_date >= m.join_date
	)
	SELECT s.customer_id, s.order_date, m2.product_name 
	FROM member_sales_cte AS s
	JOIN menu AS m2
	   ON s.product_id = m2.product_id
	WHERE rank = 1
 ````
  #### Answer:
| customer_id | order_date  | product_name |
| ----------- | ---------- |----------  |
| A           | 2021-01-07 | curry        |
| B           | 2021-01-11 | sushi        |

***

### 7. Which item was purchased just before the customer became a member?

````sql
WITH member_sales_cte AS 
	(
	   SELECT s.customer_id, m.join_date, s.order_date, s.product_id,
		  DENSE_RANK() OVER(PARTITION BY s.customer_id
		  ORDER BY s.order_date) AS rank
	   FROM sales AS s
	   JOIN members AS m
		  ON s.customer_id = m.customer_id
	   WHERE s.order_date >= m.join_date
	)
SELECT s.customer_id, s.order_date, m2.product_name 
FROM member_sales_cte AS s
JOIN menu AS m2
   ON s.product_id = m2.product_id
WHERE rank = 1;
````
  #### Answer:
| customer_id | order_date  | product_name |
| ----------- | ---------- |----------  |
| A           | 2021-01-01 |  sushi        |
| A           | 2021-01-01 |  curry        |
| B           | 2021-01-04 |  sushi        |

### 8. What is the total items and amount spent for each member before they became a member?

````sql
WITH CTE_before AS
	(
	SELECT s.customer_id, product_id, join_date, order_date
	FROM sales s JOIN members m	
		 ON s.customer_id = m.customer_id
	WHERE join_date > order_date
	)
	SELECT customer_id, COUNT(DISTINCT cte.product_id) [Number Of Products], SUM (price) [Total Price]
	FROM CTE_before cte JOIN menu m
		 ON cte.product_id = m.product_id
	GROUP BY customer_id
  ````
    #### Answer:
| customer_id | unique_menu_item | total_sales |
| ----------- | ---------- |----------  |
| A           | 2 |  25       |
| B           | 2 |  40       |

***

### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier — how many points would each customer have?

````sql
 - how many points would each customer have?
	WITH CTE_points AS 
	(
	SELECT customer_id, 
		   CASE WHEN product_name LIKE '%sushi%' THEN price * 20 ELSE price * 10 END points
	FROM sales s JOIN menu m
		 ON s.product_id = m.product_id
	) 
	SELECT customer_id, SUM(points) TotalPoints
	FROM cte_points
	GROUP BY customer_id
  ````
  
  #### Answer:
| customer_id | total_points | 
| ----------- | ---------- |
| A           | 860 |
| B           | 940 |
| C           | 360 |

***

### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi — how many points do customer A and B have at the end of January?

````sql 
	FROM members
	)
	SELECT s.customer_id,
			SUM(CASE
			WHEN product_name LIKE '%sushi%' THEN price * 20  
			WHEN order_date BETWEEN join_date AND FirstWeek THEN price * 20
			ELSE price * 10
			END) AS points
	FROM CTE_7days cte JOIN sales s
		 ON cte.customer_id = s.customer_id
		 JOIN menu m
		 ON s.product_id = m.product_id
	WHERE order_date < EndOfMonth or order_date = EndOfMonth
	GROUP BY s.customer_id
````
#### Answer:
| customer_id | total_points | 
| ----------- | ---------- |
| A           | 1370 |
| B           | 820 |














