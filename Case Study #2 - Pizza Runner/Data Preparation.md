# 🍕 Case Study #2 - Pizza Runner

## :hammer_and_wrench:	 Data Preparation

#### 1. Create Temporary Table `#temp_customer_orders`
##### Tables `#temp_customer_orders`

| Column | Actions |
| -- | -- |
| order_id | no changes |
| customer_id | no changes |
| pizza_id | no changes |
| exclusions | Get rid of Null |
| extras |  Get rid of Null |
| order_time | no changes |

The Query

````sql
DROP TABLE IF EXISTS #temp_customer_orders
CREATE TABLE #temp_customer_orders (
	order_id INT,
	customer_id INT,
	pizza_id INT,
	exclusions VARCHAR(20), 
	extras VARCHAR(20),
	order_time DATETIME
	)
	INSERT INTO #temp_customer_orders
SELECT order_id, customer_id, pizza_id,
	   CASE WHEN exclusions IS NULL OR exclusions LIKE 'null' THEN ' ' ELSE exclusions END,
	   CASE WHEN extras IS NULL OR extras LIKE 'null' THEN ' ' ELSE extras END,
	   order_time
FROM customer_orders
````

#### 2. Create Temporary Table `#temp_runner_orders`






