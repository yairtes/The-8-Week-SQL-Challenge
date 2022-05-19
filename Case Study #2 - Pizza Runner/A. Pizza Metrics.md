# 🍕 Case Study #2 - Pizza Runner

## Solution - A. Pizza Metrics

### 1. How many pizzas were ordered?

````sql
SELECT COUNT(*) PizzaOrdersNUm
FROM #temp_customer_orders
````

#### Answer:

| PizzaOrdersNUm | 
| ----------- | 
| 14        |

***

### 2. How many unique customer orders were made?

````sql
SELECT COUNT(DISTINCT order_id) unique_ordered
FROM #temp_customer_orders
````
#### Answer:

|unique_ordered|
|--------------|
|     10       |

***
### 3. How many successful orders were delivered by each runner?

````sql
SELECT runner_id, COUNT(*) Successful_Orders
FROM #temp_runner_orders
WHERE distance IS NOT NULL
GROUP BY runner_id
````
#### Answer:

runner_id | Successful_Orders
-- | --
1 | 4
2 | 3
3 | 1

***
### 4. How many of each type of pizza was delivered?

````sql
SELECT pizza_name, COUNT(co.pizza_id) 'delivered'
FROM #temp_customer_orders co JOIN pizza_names pn
	 ON co.pizza_id = pn.pizza_id
	 JOIN #temp_runner_orders ro
	 ON co.order_id = ro.order_id
WHERE duration IS NOT NULL
GROUP BY pizza_name
````

#### Answer:
pizza_name | delivered
-- | --
Meatlovers | 9
Vegetarian | 3

***

### 5. How many Vegetarian and Meatlovers were ordered by each customer?**

````sql
SELECT customer_id, pizza_name, COUNT(*) '# Of Pizza Ordered'
FROM #temp_customer_orders co JOIN pizza_names pn
	 ON co.pizza_id = pn.pizza_id
GROUP BY customer_id, pizza_name
ORDER BY pizza_name

````

**Answer:**

customer_id | pizza_name| # Of Pizza Ordered
-- | -- | --
101|	Meatlovers|	2
102|  Meatlovers|	2
103|	Meatlovers|	3
104|	Meatlovers|	3
101|	Vegetarian|	1
102|	Vegetarian|	1
103|	Vegetarian|	1
105|	Vegetarian|	1

***

### 6. What was the maximum number of pizzas delivered in a single order?

````sql
SELECT TOP 1 co.order_id, COUNT(co.order_id) '# Of pizza'
FROM #temp_customer_orders co JOIN #temp_runner_orders ro
	 ON co.order_id = ro.order_id
WHERE duration IS NOT NULL
GROUP BY co.order_id
ORDER BY '# Of pizza' DESC
````

**Answer:**

order_id|# Of pizza|
|---|---------|
|4|    3    |

***

### 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

````sql
WITH CTE_changes AS
	(
	SELECT customer_id,
		CASE WHEN exclusions = ' ' AND extras = ' ' THEN 'No Change' ELSE 'at least 1 change' END AS 'Changes'
	FROM #temp_customer_orders co JOIN pizza_recipes pr
		 ON co.pizza_id = pr.pizza_id
		 JOIN #temp_runner_orders ro
		 ON ro.order_id = co.order_id
	WHERE distance IS NOT NULL
	)
	SELECT customer_id, changes, COUNT(changes) '# Of Changes'
	FROM CTE_changes
	GROUP BY customer_id,changes
	 
````

**Answer:**

customer_id | changes| # Of Changes
-- | -- | --
101	|No Change	|2
102	|No Change	|3
103	|at least 1 change	|3
104	|at least 1 change	|2
104	|No Change	|1
105	|at least 1 change	|1

***

### 8. How many pizzas were delivered that had both exclusions and extras?

````sql
SELECT 
	SUM(CASE WHEN exclusions <> ' ' AND extras <> ' ' THEN 1 ELSE 0 END) AS 'HasBoth'
FROM #temp_customer_orders co JOIN pizza_recipes pr
	ON co.pizza_id = pr.pizza_id
	JOIN #temp_runner_orders ro
	ON ro.order_id = co.order_id
WHERE distance IS NOT NULL 
````

**Answer:**

|HasBoth|
|---------|
|    1    |

***

### 9. What was the total volume of pizzas ordered for each hour of the day?

````sql
SELECT DATEPART(hour,order_time) AS Hour, COUNT(order_id) '# Of Pizza Orderd'
FROM #temp_customer_orders co JOIN pizza_recipes pr
	ON co.pizza_id = pr.pizza_id
GROUP BY DATEPART(hour,order_time)
````

**Answer:**

Hour | # Of Pizza Orderd
-- | --
11 | 1
13 | 3
18 | 3
19 | 1
21 | 3
23 | 3

***

### 10. What was the volume of orders for each day of the week?

````sql
SELECT DATENAME(WEEKDAY,order_time) AS 'Day Of Week', COUNT(order_id) '# Of Orders'
FROM #temp_customer_orders co JOIN pizza_recipes pr
	ON co.pizza_id = pr.pizza_id
GROUP BY DATENAME(WEEKDAY,order_time)
````

**Answer:**

Day Of Week | # Of Orders
-- | --
Friday	|1
Saturday	|5
Thursday	|3
Wednesday	|5
