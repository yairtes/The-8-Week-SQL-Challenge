
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

### 5. What was the difference between the longest and shortest delivery times for all orders?


````sql
WITH CTE_distance AS
	(
	SELECT MAX(duration) AS LongestDuration, MIN(duration) AS ShortestDuration
	FROM  #temp_runner_orders ro
	WHERE duration IS NOT NULL
	)
	SELECT LongestDuration - ShortestDuration AS DifferenceBetweenDurations
	FROM CTE_distance
````
#### Answer:

| DifferenceBetweenDurations |
| -- |
| 30 |

***

### 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?

````sql
SELECT runner_id, co.order_id,
	   COUNT(pizza_id) AS Pizza_count,
	   CAST(ROUND(ro.distance/ro.duration * 60, 1) AS VARCHAR) + ' k/m' AS speed
FROM #temp_customer_orders co JOIN #temp_runner_orders ro
	 ON co.order_id = ro.order_id
WHERE duration IS NOT NULL
GROUP BY runner_id, co.order_id, ro.distance,ro.duration
````


#### Answer:
runner_id | order_id | pizza_count | speed
-- | -- | -- | --
1 | 1 | 1   | 37.5
2 | 1 | 1 | 44.44
3 | 1 | 2  | 40.2
4 | 2 | 3  | 35.1
5 | 3 | 1  | 40
7 | 2 | 1  | 60
8 | 2 | 1  | 93.6
10 | 1 | 2 | 60


***

### 7. What is the successful delivery percentage for each runner?

````sql
SELECT runner_id,
	CAST(CAST(SUM(CASE WHEN distance IS NOT NULL THEN 1 ELSE 0 END) AS FLOAT)
	/ COUNT(*) * 100 AS VARCHAR)+'%' AS SuccessfulPrecentage
FROM #temp_runner_orders
GROUP BY runner_id

````


#### Answer:
runner_id | SuccessfulPrecentage
-- | --
1 | 100
2 | 75
3 | 50


***

### Links :link:

For any Questions, comments or better solutions, feel free to contact me on my [LinkedIn account](https://www.linkedin.com/in/yair-teshuva/).<br/>
To the next solution, Pricing and Ratings click [here]([https://github.com/yairtes/8-Week-SQL-Challenge/tree/main/Case%20Study%20%232%20-%20Pizza%20Runner](https://github.com/yairtes/The-8-Week-SQL-Challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/D.%20Pricing%20and%20Ratings.md)).<br/>
To the 8-Week-Challenge site click [here](https://8weeksqlchallenge.com/case-study-1/)

