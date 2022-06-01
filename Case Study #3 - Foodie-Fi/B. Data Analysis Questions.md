
# :avocado: Case Study #3 - Foodie-Fi :avocado:

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




