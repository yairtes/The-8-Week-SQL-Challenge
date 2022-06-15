# :bank: Case Study #4: Data Bank :dollar:		

## Solution B. Customer Transactions

### 1. What is the unique count and total amount for each transaction type?

````sql
SELECT txn_type, COUNT(*) transaction_cnt, SUM(txn_amount) total_transactions
FROM customer_transactions
GROUP BY txn_type
````

#### Answer:
txn_type | transaction_cnt |  total_transactions
-- | -- | --
withdrawal	|1580	|793003
deposit	        |2671	|1359168
purchase	|1617	|806537


***

### 2. What is the average total historical deposit counts and amounts for all customers?

````sql
SELECT AVG(avg_transaction_amount) avg_transaction_amount, AVG(transaction_cnt) transaction_cnt
FROM (
		SELECT customer_id, txn_type, AVG(txn_amount) avg_transaction_amount, COUNT(*) transaction_cnt
		FROM customer_transactions
		WHERE txn_type = 'deposit'
		GROUP BY customer_id, txn_type
	 ) t
````
#### Answer:

avg_transaction_amount | transaction_cnt 
-- | -- 
508	|5

***
### 3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?

````sql
WITH CTE_cases AS
	(
	 SELECT DATENAME(MONTH,txn_date) AS month_name, customer_id,
	  	    SUM(CASE WHEN txn_type = 'deposit' THEN 1 ELSE 0 END) AS 'deposit',
		    SUM(CASE WHEN txn_type = 'purchase' THEN 1 ELSE 0 END) AS 'purchase',
		    SUM(CASE WHEN txn_type = 'withdrawal' THEN 1 ELSE 0 END) AS 'withdrawal'
	 FROM customer_transactions
	 GROUP BY DATENAME(MONTH,txn_date), customer_id
	)
	SELECT DISTINCT month_name, COUNT(*) customer_cnt
	FROM CTE_cases
	WHERE deposit > 1 AND (purchase >= 1 OR withdrawal >= 1)
	GROUP BY month_name
````
#### Answer:

month_name | customer_cnt 
-- | -- 
April	|70
February|	181
January|	168
March|	192

### 4. What is the closing balance for each customer at the end of the month?

````sql
WITH CTE_minus AS
		(
		SELECT customer_id, txn_date,
			   LEAD(txn_date,1) OVER (PARTITION BY customer_id ORDER BY txn_date) AS consecutive_month,
		 	   CASE WHEN txn_type = 'purchase' THEN - txn_amount
				    WHEN  txn_type = 'withdrawal' THEN - txn_amount
					WHEN txn_type = 'deposit' THEN txn_amount
			   END AS new_amount,
			   DATENAME(mm,txn_date) month_name
		 FROM customer_transactions
		 ),
		 CTE_sum AS
		 (
		 SELECT *,
			    SUM(new_amount) OVER(PARTITION BY customer_id ORDER BY txn_date 
								 ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS closing_balance,
				DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY month_name ) AS rank
		 FROM CTE_minus 
		 )
		 SELECT customer_id, month_name, closing_balance
		 FROM CTE_sum
		 WHERE consecutive_month is null or MONTH(txn_date) <> MONTH(consecutive_month)
````

Here is some information about [aggregated window function](https://learnsql.com/blog/sql-window-functions-rows-clause/)

#### Answer:
This is a part of the table<br/>
The full table has 1720 rows

customer_id | month_name | closing_balance
--  | -- | ---
1|	January|	312
1|	March|	-640
2|	January|	549
2|	March	|610
3|	January	|144
3|	February|	-821
3|	March	|-1222
3|	April	|-729
4|	January	|848
4|	March	|655
136|	January|	479
136|	February|	966
136|	March|	383
136|	April|	-133
137|	January|	396
137|	February|	-356
138|	January|	1316
138|	February|	320
138|	March	|75

***

### 5. What is the percentage of customers who increase their closing balance by more than 5%?

I calculated the answer for each month.<br/>
So if a customer increased he's closing balance by more than 5% for each month (january, february, march and april) the count will be 3

````sql
WITH CTE_minus AS
		(
		SELECT customer_id, txn_date,
			   LEAD(txn_date,1) OVER (PARTITION BY customer_id ORDER BY txn_date) AS consecutive_month,
		 	   CASE WHEN txn_type = 'purchase' THEN txn_amount * -1
				    WHEN  txn_type = 'withdrawal' THEN txn_amount * -1
					WHEN txn_type = 'deposit' THEN txn_amount
			   END AS new_amount,
			   DATENAME(mm,txn_date) month_name
		 FROM customer_transactions
		 ),
		 CTE_sum AS
		 (
		 SELECT *,
			    SUM(new_amount) OVER(PARTITION BY customer_id ORDER BY txn_date 
								         ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS closing_balance,
				SUM(new_amount) OVER(PARTITION BY customer_id ORDER BY txn_date 
								         ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) * 1.05 AS '5precent',
				DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY month_name ) AS rank
						  --https://learnsql.com/blog/sql-window-functions-rows-clause/
		 FROM CTE_minus 
		 ),
		 CTE_5precent AS
		 (
		 SELECT customer_id, month_name, closing_balance, [5precent],
				LEAD(closing_balance, 1) OVER(PARTITION BY customer_id ORDER BY month_name) AS next_month	
		 FROM CTE_sum
		 WHERE consecutive_month is null or MONTH(txn_date) <> MONTH(consecutive_month)
		 )
	SELECT ROUND(CAST((SELECT COUNT (*) FROM CTE_5precent WHERE [5precent] <= next_month) AS FLOAT)/ COUNT(*),2) AS 'more than 5%'
	FROM CTE_5precent 
````
#### Answer:
more than 5% | 
-- |
0.36


 
***

<p align="center">
  :nerd_face:	:nerd_face:	:nerd_face:	
</p>

### Links :link:

For any Questions, comments or better solutions, feel free to contact me on my [LinkedIn account](https://www.linkedin.com/in/yair-teshuva/).<br/>
To the next challenge solution, Data Mart click [here](https://github.com/yairtes/The-8-Week-SQL-Challenge/blob/main/Case%20Study%20%235%20-%20Data%20Mart/1.%20Data%20Cleansing%20Steps.md)<br/>
To the 8-Week-Challenge site click [here](https://8weeksqlchallenge.com/case-study-1/)

