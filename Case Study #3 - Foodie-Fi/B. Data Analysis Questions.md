
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
WITH CTE_plans AS
	(
	 SELECT customer_id, p.plan_id, plan_name,
	   	    RANK() OVER(PARTITION BY customer_id ORDER BY p.plan_id) Rank
	 FROM subscriptions s JOIN plans p
		  ON s.plan_id = p.plan_id
	)
	SELECT plan_name,
		   COUNT (DISTINCT customer_id) Number_Of_Customers,
		   CAST(CAST(COUNT(DISTINCT customer_id) AS FLOAT) / 
		   (SELECT COUNT(DISTINCT customer_id) FROM subscriptions) * 100 AS VARCHAR) +'%' AS '%'
	FROM CTE_plans
	WHERE rank = 2
	GROUP BY plan_name
	ORDER BY Number_Of_Customers DESC
````
#### Answer:

plan_name | Number_Of_Customers | %
-- | -- | --
basic monthly | 546 | 54.6
pro monthly | 325 | 32.5
pro annual | 37 | 3.7
churn | 92 | 9.2

***

### 7. What is the customer count and percentage breakdown of all 5 `plan_name` values at 2020-12-31?

````sql
WITH CTE_rankings AS
		(	
		SELECT customer_id, p.plan_id, plan_name, start_date, 
			   LEAD(start_date) OVER(PARTITION BY customer_id ORDER BY start_date, p.plan_id) Lead_Date
		FROM subscriptions s JOIN plans p
		     ON s.plan_id = p.plan_id
		WHERE start_date <= '2020-12-31'
		)
		SELECT plan_id,
			   COUNT(DISTINCT customer_id) AS Total_Customers,
			   CAST(CAST(COUNT(DISTINCT customer_id) AS FLOAT) / 
			   (SELECT COUNT(DISTINCT customer_id) FROM subscriptions) * 100 AS VARCHAR) +'%' AS '%'
		FROM CTE_rankings
		WHERE plan_id <> 0 AND Lead_Date IS NULL
		GROUP BY plan_id	
````

##### I used the LEAD function to retrive a consist value (which in our case is NULL) that will represent the last date for each customer.
		 
#### Answer:

plan_id | Total_Customers | %
-- | -- | --
1|	224|	22.4%
2|	326|	32.6%
3|	195|	19.5%
4|	236|	23.6%

***

### 8. How many customers have upgraded to an annual plan in 2020?

````sql
SELECT COUNT(DISTINCT customer_id) Customer_Num
FROM subscriptions
WHERE plan_id = 3 AND YEAR(start_date) = '2020' 
````
#### Answer:

Customer_Num 
-- |
195	

***

### 9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?

````sql
WITH CTE_Anual_Plan AS
	(
	SELECT DATEDIFF(DAY,trial.start_date, anual.start_date) number_of_days 
	FROM (SELECT customer_id, plan_id, start_date FROM subscriptions WHERE plan_id = 0) trial
		 JOIN
		 (SELECT customer_id, plan_id, start_date FROM subscriptions WHERE plan_id = 3) anual
		 ON trial.customer_id = anual.customer_id
	)
	SELECT AVG(number_of_days) days_to_anual
	FROM CTE_Anual_Plan
````

#### Answer:

days_to_anual 
-- |
104	

***

### 10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)

````sql
WITH CTE_trial AS
	(
	 SELECT customer_id, s.plan_id, start_date AS trial
	 FROM subscriptions s JOIN plans p
		  ON s.plan_id = p.plan_id
	WHERE p.plan_id = 0 -- Retrives all the dates of 'trial'
	),

	CTE_annual_plan AS
	( 
	SELECT customer_id, s.plan_id, start_date AS annual
	FROM subscriptions s JOIN plans p
		 ON s.plan_id = p.plan_id
	WHERE p.plan_id = 3 -- Retrives all the dates of 'pro annual'
	),

	CTE_date_difference AS
	(  
	SELECT DATEDIFF(DAY,t.trial,a.annual) days_to_annual
	FROM CTE_trial t JOIN CTE_annual_plan a
		 ON t.customer_id = a.customer_id -- Retrives the number of days from trial to pro annual plan based on the two first cte's
	),
	
	CTE_Groups AS
	(
	SELECT *,
		   days_to_annual/30 groups -- Dividing the dyas by 30 in order to get a column that can be concatenated the strings in the next CTE 
	FROM CTE_date_difference 
	)
	SELECT  
		   CONCAT((groups * 30) + 1, ' - ', (groups + 1) * 30, ' days') number_of_days,
		   COUNT(days_to_annual) Total_customers
	FROM CTE_Groups
	GROUP BY groups
````
#### Answer:

rating_id | order_id|
-- | -- | 
1 - 30 days	|48
31 - 60 days	|25
61 - 90 days	|33
91 - 120 days	|35
121 - 150 days	|43
151 - 180 days	|35
181 - 210 days	|27
211 - 240 days	|4
241 - 270 days	|5
271 - 300 days	|1
301 - 330 days	|1
331 - 360 days	|1

***

### 11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?

````sql
WITH CTE_pro_monthly AS
	(
	 SELECT customer_id, s.plan_id, start_date AS basic_monhly
	 FROM subscriptions s JOIN plans p
		  ON s.plan_id = p.plan_id
	WHERE p.plan_id = 1 AND
		  YEAR(start_date) = '2020'
	
	),

	 CTE_annual_plan AS
	( 
	SELECT customer_id, s.plan_id, start_date AS pro_monhly
	FROM subscriptions s JOIN plans p
		 ON s.plan_id = p.plan_id
	WHERE p.plan_id = 2 AND
		  YEAR(start_date) = '2020'
	)

	SELECT COUNT(*) Number_Of_downgrades
	FROM CTE_pro_monthly pm JOIN CTE_annual_plan ap
		 ON pm.customer_id = ap.customer_id
	WHERE pro_monhly < basic_monhly
````
#### Answer:

Number_Of_downgrades | 
-- |
0

***

### Links :link:

For any Questions, comments or better solutions, feel free to contact me on my [LinkedIn account](https://www.linkedin.com/in/yair-teshuva/).<br/>
To the next solution, C. Challenge Payment Questions click [here](https://github.com/yairtes/The-8-Week-SQL-Challenge/blob/main/Case%20Study%20%233%20-%20Foodie-Fi/C.%20Challenge%20Payment%20Question.md).<br/>
To the 8-Week-Challenge site click [here](https://8weeksqlchallenge.com/case-study-1/)
