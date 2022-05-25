
# 🍕 Case Study #2 - Pizza Runner

## Solution B.Runner And Customer Experience

### 1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

````sql
SELECT 
  DATEPART(WEEK, registration_date) AS registration_week,
  COUNT(runner_id) AS runner_sign
FROM runners
GROUP BY DATEPART(WEEK, registration_date)
````

#### Answer:

weeks | total
-- | --
1 | 2
2 | 1
3 | 1

***

### 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

````sql
SELECT runner_id, AVG(DATEDIFF(MINUTE, order_time, pickup_time)) AS AvgTime
FROM #temp_customer_orders co JOIN #temp_runner_orders ro
	 ON co.order_id = ro.order_id
WHERE duration IS NOT NULL
GROUP BY runner_id
````
#### Answer:

runner_id | AvgTime
-- | --
1 | 15
2 | 24
3 | 10

***
### 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?


````sql
WITH CTE_Pizza_amount AS
	(
	SELECT co.order_id, AVG(DATEDIFF(MINUTE, order_time, pickup_time)) AS AvgTime, COUNT(co.order_id) '# Of Pizzas'
	FROM #temp_customer_orders co JOIN #temp_runner_orders ro
		 ON co.order_id = ro.order_id
	WHERE duration IS NOT NULL
	GROUP BY co.order_id
	)
	SELECT [# Of Pizzas], AVG(avgtime) AvgMakingTime
	FROM CTE_Pizza_amount
	GROUP BY [# Of Pizzas]
````
#### Answer:

#Of Pizzas | AvgTime
-- | --
1 | 12
2 | 18
3 | 30

##### For 1 pizza Avg of 12 Minutes making per pizza <br/>
##### For 2 pizza Avg of 9 Minutes making per pizza <br/>
##### For 3 pizza Avg of 10 Minutes making per pizza <br/>
##### So the ulimate efficiency rate is for 2 pizza's <br/>
