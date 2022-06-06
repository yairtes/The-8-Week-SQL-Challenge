
# :avocado: Case Study #3 - Foodie-Fi :avocado:

## Solution D. Pricing and Ratings

### The Foodie-Fi team wants you to create a new `payments` table for the year 2020 that includes amounts paid by each customer in the `subscriptions` table with the following requirements:

- monthly payments always occur on the same day of month as the original `start_date` of any monthly paid plan
- upgrades from basic to monthly or pro plans are reduced by the current paid amount in that month and start immediately
- upgrades from pro monthly to pro annual are paid at the end of the current billing period and also starts at the end of the month period
- once a customer churns they will no longer make payments


````sql
						-- First CTE - Building the basic table plus a Lead function
WITH CTE_lead AS
			(
			 SELECT s.*, plan_name, price,
					LEAD(start_date) OVER(PARTITION BY customer_id ORDER BY p.plan_id) rank
			 FROM subscriptions s JOIN plans p
				  ON s.plan_id = p.plan_id
			),  
						-- Changing the Null's and the 2021 dates to the highest date on 2020
	CTE_case_statemant AS
			( 
			SELECT * ,
				  CASE WHEN rank IS NULL or rank > '2020-12-31' THEN '2020-12-31' ELSE rank END next_date
			FROM CTE_lead
			), 
						-- Creating a new column of the previous month in order to get the end of the payments in the recursive CTE

	CTE_previous_month AS
			(
			SELECT *,
			DATEADD(MONTH, -1, next_date) previous_month
			from CTE_case_statemant
			),
						-- Creating the recursive CTE with the DATEADD func in order to get all the monthes that have been paid until meeting 2 conditions
					
	CTE_recursive_date AS
			( 
			  SELECT customer_id,
					 plan_id,
					 plan_name,
					 start_date,
					 price,
					 next_date,
					 previous_month
			  FROM CTE_previous_month
			  UNION ALL
			  SELECT customer_id,
					 plan_id,
					 plan_name,
					 DATEADD(MONTH,1,start_date), 
					 price,
					 next_date,
					 previous_month
			  FROM CTE_recursive_date
			  WHERE plan_name <> 'pro annual' AND
					start_date < previous_month 
			),
	CTE_last AS
			(
			SELECT customer_id,
				   plan_id,
				   plan_name,
				   start_date AS payment_date,
				   price AS amount, 
				   RANK() OVER (PARTITION BY customer_id ORDER BY start_date) AS payment_order
			FROM CTE_recursive_date
			WHERE YEAR(start_date) = '2020' AND
				  plan_name <> 'trial' AND
				  plan_name <> 'churn'  
				  )
SELECT *
FROM CTE_last
ORDER BY customer_id, plan_id, payment_date
````

#### Answer:

customer_id|plan_id|plan_name|payment_date|amount|payment_order
--|--|--|--|--|--|
1|	1	|basic monthly|	2020-08-08	|9.90|	1
1	|1	|basic monthly|	2020-09-08	|9.90	|2
1	|1	|basic monthly|	2020-10-08	|9.90	|3
1	|1	|basic monthly|	2020-11-08	|9.90	|4
1	|1	|basic monthly|	2020-12-08	|9.90	|5
2	|3	|pro annual|	2020-09-27	|199.00	|1
3	|1	|basic monthly|	2020-01-20	|9.90	|1
3	|1	|basic monthly|	2020-02-20	|9.90	|2
3	|1	|basic monthly|	2020-03-20	|9.90	|3
3	|1	|basic monthly|	2020-04-20|	9.90	|4
3	|1	|basic monthly|	2020-05-20	|9.90	|5
3	|1	|basic monthly|	2020-06-20|	9.90	|6


- I didn't manage to complete the second condition `upgrades from basic to monthly or pro plans are reduced by the current paid amount in that month and start immediately`. So I will be happy to learn about a way to answer this part*
- I learned about recursive CTE's from [this](https://www.youtube.com/watch?v=7hZYh9qXxe4&t=1s) Youtube video

***
<p align="center">
  :nerd_face:	:nerd_face:	:nerd_face:	
</p>

### Links :link:

For any Questions, comments or better solutions, feel free to contact me on my [LinkedIn account](https://www.linkedin.com/in/yair-teshuva/).<br/>
To the next solution, Case Study #4 - Data Bank click [here](https://github.com/yairtes/The-8-Week-SQL-Challenge/blob/main/Case%20Study%20%234%20-%20Data%20Bank/A.%20Customer%20Nodes%20Exploration.md).<br/>
To the 8-Week-Challenge site click [here](https://8weeksqlchallenge.com/case-study-1/)
