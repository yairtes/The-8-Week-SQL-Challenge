
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
SELECT 
	CAST(SUM(
		CASE WHEN pizza_id = 1 THEN 12 
		WHEN pizza_id = 2 THEN 10
		END
		) +
		SUM(CEILING(CAST(LEN(REPLACE(extras,',','')) AS FLOAT)/2)) AS VARCHAR) + '$' AS Total_money_made
FROM #temp_customer_orders co JOIN #temp_runner_orders ro
	 ON co.order_id = ro.order_id
WHERE distance  IS NOT NULL
````
#### Answer:

Total_money_made | 
-- | 
142$ |

***
### 3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.


````sql
DROP TABLE IF EXISTS runner_review
CREATE TABLE runner_review (
	rating_id INT IDENTITY(1,1) PRIMARY KEY,
	order_id INT,
	runner_id INT,
	rating INT,
	reviews VARCHAR(MAX),
	rating_time TIMESTAMP
	)
	SET IDENTITY_INSERT runner_review ON
	INSERT INTO runner_review 
		(
		 rating_id,order_id,runner_id,rating,reviews,rating_time
		)
		VALUES
			  ('1', '2', '1', '3','Not bad',default),
			  ('2', '3', '1', '4','Kind runner', default),
			  ('3', '4', '2', '5','fast delivery', default),
			  ('4', '7', '2', '5','Got on time', default),
			  ('5', '5', '3', '2','The was cold', default),
			  ('6', '8', '2', '3','lol', default),
			  ('7', '9', '2', '5','Great pizza, great service - see you tomorrow', default),
			  ('8', '10', '1', '2','Lame', default)
	SET IDENTITY_INSERT runner_review OFF
	SELECT * FROM runner_review
````
#### Answer:

rating_id | order_id|runner_id|rating|review|rating_time
-- | -- | -- | --|--|--
1|	2|	1|	3|	Not bad|	 2020-01-01 19:10:54.000
2|	3|	1|	4|	Kind runner|	 2020-01-01 19:10:54.000
3|	4|	2|	5|	fast delivery|	 2020-01-01 19:10:54.000
4|	7|	2|	5|	Got on time|	 2020-01-01 19:10:54.000
5|	5|	3|	2|	The was cold|	 2020-01-01 19:10:54.000
6|	8|	2|	3|	lol|	 2020-01-01 19:10:54.000
7|	9|	2|	5|	Great pizza, great service - see you tomorrow|	 2020-01-01 19:10:54.000
8|	10|	1|	2|	Lame|	 2020-01-01 19:10:54.000


### 4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?


````sql
SELECT customer_id,
	   co.order_id,
	   rating, 
	   reviews,
	   rating_time,
	   order_time,
	   pickup_time,
	   DATEDIFF(MINUTE,order_time,pickup_time) AS Time_from_order_to_pickup,
	   duration AS Delivery_duration,
	   ROUND(distance/duration * 60,1) AS average_speed,
	   COUNT(pizza_id) AS Total_number_of_pizzas
FROM #temp_customer_orders co JOIN #temp_runner_orders ro
	 ON co.order_id = ro.order_id
	 JOIN runner_review rr
	 ON ro.order_id = rr.order_id
WHERE duration IS NOT NULL
GROUP BY customer_id,
		 co.order_id,
		 rating,
		 reviews,
		 rating_time,
		 pickup_time,
		 order_time,
		 DATEDIFF(MINUTE,order_time,pickup_time),
		 ROUND(distance/duration * 60,1),
		 duration
````
#### Answer:
csutomer_id|order_id|ratin_id|review|rating_date|order_time|pickup_time|delivery_duration|average_speed|Total_number_of_pizzas
-- | -- | -- | --|--|--|--|--|--|--
101|	2|	3|	Not bad|	2020-01-01 19:10:54.000|	2020-01-01 19:00:52.000|	2020-01-01 19:10:54.000	|10	|27	|44.4	|1
102|	3|	4|	Kind runner|	2020-01-01 19:10:54.000|	2020-01-02 23:51:23.000|	2020-01-03 00:12:37.000	|21	|20	|40.2	|2
102|	8|	3|	lol|	2020-01-01 19:10:54.000	|2020-01-09 23:54:33.000	|2020-01-10 00:15:02.000	|21	|15	|0.9	|93.6|1
103|	4|	5|	fast delivery|	2020-01-01 19:10:54.000|	2020-01-04 13:23:46.000	|2020-01-04 13:53:03.000	|30	|40	|35.1	|3
104|	5|	2|	The was cold|	2020-01-01 19:10:54.000|	2020-01-08 21:00:29.000	|2020-01-08 21:10:57.000	|10	|15	|40	|1
104|	10|	2|	Lame|	2020-01-01 19:10:54.000	|2020-01-11 18:34:49.000	|2020-01-11 18:50:20.000	|16	|10	|0.9	|60|2
105|	7|	5|	Got on time|	2020-01-01 19:10:54.000|	2020-01-08 21:20:29.000|	2020-01-08 21:30:45.000	|10	|25	|60	|1

***

### 5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?

````sql
SELECT 
	SUM(
		CASE WHEN pizza_id = 1 THEN 12 
		WHEN pizza_id = 2 THEN 10
		END)
		-
		(SELECT SUM(distance * 0.30) FROM #temp_runner_orders) AS with_this_amount_of_money_I_should_open_a_hotdog_store
FROM #temp_customer_orders co JOIN #temp_runner_orders ro
	 ON co.order_id = ro.order_id
WHERE distance  IS NOT NULL
````


#### Answer:

with_this_amount_of_money_I_should_open_a_hotdog_store | 
-- | 
94.44 |

***

<p align="center">
  :nerd_face:	:nerd_face:	:nerd_face:	
</p>

### Links :link:

For any Questions, comments or better solutions, feel free to contact me on my [LinkedIn account](https://www.linkedin.com/in/yair-teshuva/).<br/>
To the next challenge solution, Foodie-fi click [here](https://github.com/yairtes/8-Week-SQL-Challenge/tree/main/Case%20Study%20%232%20-%20Pizza%20Runner).<br/>
To the 8-Week-Challenge site click [here](https://8weeksqlchallenge.com/case-study-1/)

