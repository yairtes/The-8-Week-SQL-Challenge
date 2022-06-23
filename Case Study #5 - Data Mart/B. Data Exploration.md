# :shopping_cart:	 Case Study #4: Data Mart :shopping_cart:		

## Solution B. Data Exploration

### 1. What day of the week is used for each `week_date` value?


````sql
SELECT DISTINCT week_date, DATENAME(WEEKDAY, week_date) 'weekday'
FROM clean_weekly_sales
````
NOTE: Not all ouptut is displayed, be sure you have 72 rows.

#### Answer:

week_date | weekday
-- | --
2019-04-22	|Monday
2019-05-06	|Monday
2018-07-30	|Monday
2020-06-08	|Monday
2018-08-13	|Monday
...|...

The answer is obviously *Monday*

***

### 2. What range of week numbers are missing from the dataset?

In the first step I generated numbers from 1 to 52 (number of weeks in a year) with a recursive CTE
In th second step I used EXCEPT to find which weeks are missing fro the dataset.

````sql
WITH u AS
	(
     SELECT 1 AS n
     UNION ALL
     SELECT n + 1
     FROM u
     WHERE n < 52
	)
  SELECT * FROM u
  EXCEPT
  SELECT DISTINCT week_number FROM clean_weekly_sales
````
Note that in [this link](https://docs.microsoft.com/en-us/sql/t-sql/functions/generate-series-transact-sql?view=sql-server-ver16) it says that MSSQL 2022 will support a simple GENERATE_SERIES function, which will make the recursive CTE Unnecessary for generating serieses.

NOTE: Not all ouptut is displayed, be sure you have 28 rows.

#### Answer:

n | 
-- |
1
2
3
4
5
6
7
8
9
10
11
12
37
...


***

### 3. How many total transactions were there for each year in the dataset?


````sql
SELECT year, SUM(transactions) total_transactions
FROM clean_weekly_sales
GROUP BY year
ORDER BY year
````
#### Answer:

year | total_transactions 
-- | -- 
2018	|346406460
2019	|365639285
2020	|375813651

***

### 4. What is the total sales for each region for each month?

````sql
SELECT region, MONTH(week_date) AS month, SUM(CAST(sales AS FLOAT)) total_sales
FROM clean_weekly_sales
GROUP BY region, MONTH(week_date)
ORDER BY region, MONTH(week_date)
````

NOTE: Not all ouptut is displayed, be sure you have 49 rows.

#### Answer:

region | month | total_sales
--  | -- | --
AFRICA|	3|	567767480
AFRICA|	4|	1911783504
AFRICA|	5|	1647244738
AFRICA|	6|	1767559760
...| ...| ...
ASIA|	3|	529770793
ASIA|	4|	1804628707
ASIA|	5|	1526285399
...|...|...


***

### 5. What is the total count of transactions for each platform

````sql
SELECT region, COUNT(transactions) transaction_cnt
FROM clean_weekly_sales
GROUP BY region
````

#### Answer:

region | month 
--  | -- 
OCEANIA|	2448
EUROPE|	2436
SOUTH |AMERICA	2441
AFRICA|	2448
CANADA|	2448
ASIA	|2448
USA	|2448

*** 


### 6. What is the percentage of sales for Retail vs Shopify for each month?

````sql
SELECT 'retail' AS platform,ROUND((SELECT SUM(CAST(sales AS FLOAT))  
	   FROM clean_weekly_sales
	   WHERE platform = 'retail')
	   /
	   	SUM(CAST(sales AS FLOAT)) * 100, 2) sales_precentage
FROM clean_weekly_sales 
UNION
SELECT 'shopify' AS platform, ROUND((SELECT SUM(CAST(sales AS FLOAT)) 
	   FROM clean_weekly_sales
	   WHERE platform = 'shopify')
	   /
	   	SUM(CAST(sales AS FLOAT)) * 100,2) sales_precentage
FROM clean_weekly_sales 


 ---- second approach ----

 SELECT platform, month_number,
	ROUND
	(SUM(CAST(sales AS FLOAT)) * 100.0 
	/
	SUM(SUM(CAST(sales AS FLOAT))) over()
	,2) AS sales_precentage
	FROM clean_weekly_sales
	GROUP BY platform, month_number
	ORDER BY platform, month_number
````

NOTE: Not all ouptut is displayed, be sure you have 49 rows.

#### Answer:

##### The first approach will return this answer:

platform | sales_percentage
--|--
retail	| 97.33
shopify	| 2.67


##### The second one will return this answer:

platform | month_number | sales_precentage
--  | -- | --
Retail|	3|	5.64
Retail|	4|	18.99
Retail|	5|	16.16
Retail|	6|	17.3
Retail|	7|	18.87
Retail|	8|	17.65
Retail|	9|	2.71
Shopify|	3|	0.14
Shopify|	4|	0.47
Shopify|	5|	0.45
Shopify|	6|	0.49
Shopify|	7|	0.53
Shopify|	8|	0.53
Shopify|	9|	0.07
*** 

### 7. What is the percentage of sales by demographic for each year in the dataset?

````sql
SELECT demographic, 
	   year,
	   ROUND(
	   SUM(CAST(sales AS FLOAT)) * 100.0 
	   /
	   SUM(SUM(CAST(sales AS FLOAT))) over()
	   ,2) sales_precentage
FROM clean_weekly_sales
GROUP BY demographic, year
ORDER BY demographic, year
````


#### Answer:

demographic | year | sales_precentage
--  | -- | --
Couples|	2018|	8.35
Couples|	2019|	9.2
Couples|	2020|	9.94
families|	2018|	10.13
families|	2019|	10.96
families|	2020|	11.33
unknown|	2018|	13.18
unknown|	2019|	13.58
unknown|	2020|	13.34

*** 

### 8. Which `age_band` and `demographic` values contribute the most to Retail sales?

````sql
SELECT demographic, 
	   age_band,
	   ROUND(
	   SUM(CAST(sales AS FLOAT)) * 100.0 
	   /
	   SUM(SUM(CAST(sales AS FLOAT))) over()
	   ,2) sales_precentage
FROM clean_weekly_sales
WHERE platform = 'retail'
GROUP BY demographic, age_band
ORDER BY sales_precentage DESC
````

#### Answer:

demographic | age_band | sales_precentage
--  | -- | --
unknown|	unknown	|40.52
families|	Retirees	|16.73
Couples|	Retirees	|16.07
families|	Middle Aged|	10.98
Couples|	young adults|	6.56
Couples|	Middle Aged	|4.68
families|	young adults|	4.47

*** 

### 9. Can we use the `avg_transaction` column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?

Let's check.<br/>
1. We will calculate the average with the avg_transaction column<br/>
2. We will sum and devide the sales from the transactions and group it with the year and platform <br/>
the second aprouch is the safe one, and we want to see if the results are identical.

````sql
SELECT 
  year, 
  platform, 
  ROUND(AVG(avg_transaction),0) AS with_avg_transaction, 
  ROUND(SUM(CAST(sales AS FLOAT)) / SUM(CAST(transactions AS FLOAT)), 2) AS avg_transaction_group
FROM clean_weekly_sales
GROUP BY year, platform
ORDER BY year, platform
````

#### Answer:

year | platform | with_avg_transaction |  avg_transaction_group
--  | -- | -- | --
2018|	Retail	|43	|36.56
2018|	Shopify	|188	|192.48
2019|	Retail	|42	|36.83
2019|	Shopify	|178	|183.36
2020|	Retail	|41	|36.56
2020|	Shopify	|175	|179.03

The answers are not the same, so the `avg_transaction_group` aprouch is the right one.

***

<p align="center">
  :nerd_face:	:nerd_face:	:nerd_face:	
</p>

### Links :link:

For any Questions, comments or better solutions, feel free to contact me on my [LinkedIn account](https://www.linkedin.com/in/yair-teshuva/).<br/>
To the next solution, c. Before & After Analysis click [here](https://github.com/yairtes/The-8-Week-SQL-Challenge/blob/main/Case%20Study%20%235%20-%20Data%20Mart/C.%20Before%20%26%20After%20Analysis.md)<br/>
To the 8-Week-Challenge site click [here](https://8weeksqlchallenge.com/case-study-1/)
