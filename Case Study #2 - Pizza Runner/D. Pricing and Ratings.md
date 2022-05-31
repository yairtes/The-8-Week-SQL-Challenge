
# 🍕 Case Study #2 - Pizza Runner

## Solution D. Pricing and Ratings

### 1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?

````sql
SELECT 
		CAST(SUM(
			CASE WHEN pizza_id = 1 THEN 12 
			WHEN pizza_id = 2 THEN 10
			END
			) AS VARCHAR) + '$' AS Total_money_made
FROM #temp_customer_orders co JOIN #temp_runner_orders ro
	 ON co.order_id = ro.order_id
WHERE distance  IS NOT NULL

````

#### Answer:

Total_money_made | 
-- | 
138$ |


***

### 2. What if there was an additional $1 charge for any pizza extras? 

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
### 3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.


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

### 4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?


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

### 5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?

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
To the next chalenge solution, Foodie-fi click [here](https://github.com/yairtes/8-Week-SQL-Challenge/tree/main/Case%20Study%20%232%20-%20Pizza%20Runner).<br/>
To the 8-Week-Challenge site click [here](https://8weeksqlchallenge.com/case-study-1/)

