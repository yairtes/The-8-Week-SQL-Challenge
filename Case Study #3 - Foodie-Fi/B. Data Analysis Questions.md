
# :avocado: Case Study #3 - Foodie-Fi :avocado:

## Solution D. Pricing and Ratings

### 1. How many customers has Foodie-Fi ever had?


````sql
SELECT COUNT(DISTINCT customer_id) AS UniqueCustomers
FROM subscriptions;
````

#### Answer:

UniqueCustomers | 
-- | 
1000 |


***

### 2. What is the monthly distribution of `trial` plan `start_date` values for our dataset - use the start of the month as the group by value


````sql
SELECT
	MONTH(start_date) AS Month_Number,
	DATENAME(MONTH,start_date) Month_Name,
	COUNT(p.plan_id) AS Number_Of_Trials
FROM subscriptions s JOIN plans p
	 ON s.plan_id = p.plan_id
WHERE p.plan_id = 0
GROUP BY MONTH(start_date) ,
		 DATENAME(MONTH,start_date) 
ORDER BY Month_Number
````
#### Answer:

Month_Number | Month_Name | Number_Of_Trials
-- | -- | --
1 | January | 88
2 | February | 68
3 | March | 94
4 | April | 81
5 | May | 88
6 | June | 79
7 | July | 89
8 | August | 88
9 | September | 87
10 | October | 79
11 | November | 75
12 | December | 84

***
### 3. What plan `start_date` values occur after the year 2020 for our dataset? Show the breakdown by count of events for each `plan_name`


````sql
SELECT p.plan_id, plan_name, COUNT(start_date) 'Number_Of_Events'
FROM subscriptions s JOIN plans p
     ON s.plan_id = p.plan_id
WHERE YEAR(start_date) > '2020'
GROUP BY p.plan_id,plan_name
ORDER BY p.plan_id
````
#### Answer:

plan_id | plan_name | Number_Of_Events
-- | -- | --
1|	basic monthly|	8
2|	pro monthly|	60
3|	pro annual|	63
4|	churn|	71

### 4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
````sql
SELECT COUNT(*) AS 'Numbe_Of_Churnes', 
	   CAST((
			 SELECT CAST(COUNT(*) AS FLOAT)
			 FROM subscriptions s JOIN plans p
			 ON s.plan_id = p.plan_id
			 WHERE plan_name = 'churn'
			 )
			     /
				 (
				 SELECT COUNT(DISTINCT customer_id)
				 FROM subscriptions) * 100 AS VARCHAR
				 ) +'%' AS 'Churn_Precentage'
FROM subscriptions s JOIN plans p
	 ON s.plan_id = p.plan_id
WHERE plan_name = 'churn'
````
#### Answer:

Numbe_Of_Churnes |Churn_Precentage
-- | -- 
307|	30.7%

***

### 5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?


````sql
WITH CTE_trial AS
	(
	 SELECT customer_id, p.plan_id, plan_name,
	   	    RANK() OVER(PARTITION BY customer_id ORDER BY p.plan_id) Rank
	 FROM subscriptions s JOIN plans p
		  ON s.plan_id = p.plan_id
	)
	SELECT COUNT(*) Total_Churn, 
		   CAST(CAST(COUNT(DISTINCT customer_id) AS FLOAT) / 
		   (SELECT COUNT(DISTINCT customer_id) FROM subscriptions) * 100 AS VARCHAR) +'%' AS Churn_Precentage
	FROM CTE_trial
	WHERE rank = 2 AND plan_name = 'churn'
````
#### Answer:

Total_Churn | Churn_Precentage
-- | --
92 |	9.2%

 ##### There are two Assumptions:<br/>
	1. Every customer starts with plan number 0, which is trial
	2. If after the trial the customer churns immediately, he will have two steps - plan 0 and plan 4


***

### 6. What is the number and percentage of customer plans after their initial free trial?

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


### 7. What is the customer count and percentage breakdown of all 5 `plan_name` values at 2020-12-31?
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

***

### 8. How many customers have upgraded to an annual plan in 2020?

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

***

### 9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?

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

***

### 10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)

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

***

### 11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?

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

***
