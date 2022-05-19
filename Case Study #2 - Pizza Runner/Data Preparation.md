# 🍕 Case Study #2 - Pizza Runner

## :hammer_and_wrench:	 Data Preparation

#### 1. Create Temporary Table `#temp_customer_orders`
##### Table `#temp_customer_orders`

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
The new table looks like that


no | order_id | customer_id | pizza_id | exclusions | extras | order_time
-- | -- | -- | -- | -- | -- | --
1 | 1 | 101 | 1 |  |  | 2020-01-01 18:05:02.000
2 | 2 | 101 | 1 |  |  | 2020-01-01 19:00:52.000
3 | 3 | 102 | 1 |  |  | 2020-01-02 23:51:23.000
4 | 3 | 102 | 2 |  |  | 2020-01-02 23:51:23.000
5 | 4 | 103 | 1 | 4 |  | 2020-01-04 13:23:46.000
6 | 4 | 103 | 1 | 4 |  | 2020-01-04 13:23:46.000
7 | 4 | 103 | 2 | 4 |  | 2020-01-04 13:23:46.000
8 | 5 | 104 | 1 |  | 1 | 2020-01-08 21:00:29.000
9 | 6 | 101 | 2 |  |  | 2020-01-08 21:03:13.000
10 | 7 | 105 | 2 |  | 1 | 2020-01-08 21:20:29.000
11 | 8 | 102 | 1 |  |  | 2020-01-09 23:54:33.000
12 | 9 | 103 | 1 | 4 | 1, 5 | 2020-01-10 11:22:59.000
13 | 10 | 104 | 1 |  |  | 2020-01-11 18:34:49.000
14 | 10 | 104 | 1 | 2, 6 | 1, 4 | 2020-01-11 18:34:49.000

***

#### 2. Create Temporary Table `#temp_runner_orders`
##### Table `#temp_customer_orders`

| Column | Actions |
| -- | -- |
| order_id | no changes |
| runner_id | no changes |
| pickup_time | Convert the string null to a null value |
| distance | Convert the string null to a null value |
|  | Use **TRIM** to remove 'km' or ' km' |
| duration | Use CASE with conditional exclusions is NULL or 'null' then '' |
|  | Use **TRIM** to remove 'mins' or ' mins' or ' minute' or 'minutes' or ' minutes' |
| cancellation | Convert the null to a ' ' |

The Query
````sql
DROP TABLE IF EXISTS #temp_runner_orders
CREATE TABLE #temp_runner_orders (
	order_id INT,
	runner_id INT,
	pickup_time DATETIME,
	distance FLOAT,
	duration FLOAT,
	cancellation VARCHAR(50)
	)
	INSERT INTO #temp_runner_orders
	SELECT order_id, runner_id, 
		   CASE WHEN pickup_time LIKE 'null' THEN NULL ELSE pickup_time END AS pickup_time,
		   CASE 
				WHEN distance LIKE '%km' THEN TRIM('km' from distance)
				WHEN distance LIKE 'null' THEN NULL ELSE distance 
				END AS distance,
		   CASE 
				WHEN duration LIKE 'null' THEN NULL
				WHEN duration LIKE '%mins' THEN TRIM('mins' from duration)
				WHEN duration LIKE '%minute' THEN TRIM('minute' from duration)
				WHEN duration LIKE '%minutes' THEN TRIM('minutes' from duration)
				ELSE duration
				END AS duration,
		   CASE
			    WHEN cancellation IS NULL or cancellation LIKE 'null' THEN ' '
			    ELSE cancellation
			    END AS cancellation	   
FROM runner_orders
````
The new table looks like that

order_id | runner_id | pickup_time | distance | duration | cancellation
-- | -- | -- | -- | -- | --
1 | 1 | 2020-01-01 18:15:34.000 | 20 | 32 |  
2 | 1 | 2020-01-01 19:10:54.000 | 20 | 27 |  
3 | 1 | 2020-01-03 00:12:37.000 | 13.4 | 20 |  
4 | 2 | 2020-01-04 13:53:03.000 | 23.4 | 40 |  
5 | 3 | 2020-01-08 21:10:57.000 | 10 | 15 |  
6 | 3 |Null | Null | Null | Restaurant Cancellation
7 | 2 | 2020-01-08 21:30:45.000 | 25 | 25 |  
8 | 2 | 2020-01-10 00:15:02.000 | 23.4 | 15 |  
9 | 2 | Null | Null | Null | Customer Cancellation
10 | 1 | 2020-01-11 18:50:20.000 | 10 | 10 | 

*** 

#### 3. Create Temporary Table `#temp_pizza_recipes`
##### Table `#temp_pizza_recipes`

| Column | Actions |
| -- | -- |
| pizza_id | no changes |
| toppings | change datatype to **varchar** |
|  | Use **STRING_SPLIT** to split the toppings |

The Query 

````sql
DROP TABLE IF EXISTS #temp_pizza_recipes
CREATE TABLE #temp_pizza_recipes (
	pizza_id VARCHAR(50),
	toppings VARCHAR(50)
	)
INSERT INTO #temp_pizza_recipes
	SELECT pizza_id, value AS toppings
	FROM pizza_recipes
	CROSS APPLY 
	(SELECT * FROM string_split(toppings, ',')) t
````

And the new table looks like that

pizza_id | toppings
-- | -- 
1 | 1
1 | 2
1 | 3
1 | 4
1 | 5
1 | 6
1 | 8
1 | 10
2 | 4
2 | 6
2 | 7
2 | 9
2 | 11
2 | 12

***
#### 4. Change data type for `pizza_topping`
##### Table `pizza_topping`

| Column | Actions |
| -- | -- |
| topping_id | no changes |
| topping_name | change datatype to **varchar** |

````sql
ALTER TABLE pizza_toppings
ALTER COLUMN topping_id VARCHAR(50)
````

The table looks like that

topping_id | topping_name
-- | -- 
1 | Bacon
2 | BBQ Sauce
3 | Beef
4 | Cheese
5 | Chicken
6 | Mushrooms
7 | Onions
8 | Pepperoni
9 | Peppers
10 | Salami
11 | Tomatoes
12 | Tomato Sauce

***

#### For the asnwers for part A - **Pizza metrics** click [here](https://github.com/yairtes/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/A.%20Pizza%20Metrics.md)


