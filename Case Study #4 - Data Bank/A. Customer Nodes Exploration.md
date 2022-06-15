# :bank: Case Study #4: Data Bank :dollar:		

## Solution A. Customer Nodes Exploration

### 1. How many unique nodes are there on the Data Bank system?

````sql
SELECT COUNT(DISTINCT node_id) AS unique_nodes
FROM customer_nodes
````

#### Answer:

unique_nodes | 
-- | 
5 |


***

### 2.What is the number of nodes per region?

````sql
SELECT region_name, COUNT(node_id) nodes_cnt
FROM regions r JOIN customer_nodes cn
	 ON r.region_id = cn.region_id
GROUP BY region_name
````
#### Answer:

region_name | nodes_cnt 
-- | -- 
Africa	|714
America	|735
Asia	|665
Australia|	770
Europe	|616 


***
### 3. How many customers are allocated to each region?

````sql
SELECT region_name, COUNT(DISTINCT customer_id) customer_cnt
FROM regions r JOIN customer_nodes cn
	 ON r.region_id = cn.region_id
GROUP BY region_name
ORDER BY COUNT(DISTINCT customer_id)
````
#### Answer:

region_name | customer_cnt 
-- | -- 
Europe	|88
Asia	|95
Africa	|102
America|	105
Australia	|110

### 4. How many days on average are customers reallocated to a different node?

Finding the broken data in the `start_date` column:
````sql
SELECT YEAR(start_date), YEAR(end_date), COUNT(*)
FROM  customer_nodes
GROUP BY YEAR(start_date), YEAR(end_date)
````
And then ignore the misleading data:
````sql
SELECT AVG(DATEDIFF(DAY, start_date, end_date)) time_between_nodes
FROM customer_nodes
WHERE YEAR(end_date) <> '9999'
````
#### Answer:

time_between_nodes |
--  |
14

***

### 5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?


````sql
WITH CTE_time_between_nodes AS
	 (
	 SELECT cn.*,
			region_name,
			DATEDIFF(DAY, start_date, end_date) time_between_nodes
	 FROM regions r JOIN customer_nodes cn
		  ON r.region_id = cn.region_id
	 WHERE YEAR(end_date) <> '9999'
	 ),
	 CTE_ntile AS
	 (
	 SELECT *,
			NTILE(2) OVER(ORDER BY time_between_nodes ) ntile,
			PERCENTILE_CONT(0.8) WITHIN GROUP (ORDER BY time_between_nodes) 
								 OVER(PARTITION BY region_id) AS 'percentile_80',
			PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY time_between_nodes) 
								 OVER(PARTITION BY region_id) AS 'percentile_95'
	 FROM CTE_time_between_nodes 
	 ),
	 CTE_median AS
	 (
	  SELECT region_id,
			 region_name,
			 percentile_80,
			 percentile_95,
			 (SELECT MAX(time_between_nodes) FROM CTE_ntile WHERE ntile = 1) AS median
	  FROM CTE_ntile
	  GROUP BY region_id, region_name, percentile_80, percentile_95
	  )
	  SELECT * 
	  FROM CTE_median
````
#### Answer:
region_id | region_name | percentile_80 | percentile_95 | median
-- | -- | -- |-- | --
1	|Australia	|23|	28|	15
2	|America	|23	|28	|15
3	|Africa |	24	|28	|15
4	|Asia	   |23	|28	|15
5	|Europe |	24	|28|	15

 
***

<p align="center">
  :nerd_face:	:nerd_face:	:nerd_face:	
</p>

### Links :link:

For any Questions, comments or better solutions, feel free to contact me on my [LinkedIn account](https://www.linkedin.com/in/yair-teshuva/).<br/>
To the next solution, C. Challenge Payment Questions click [here](https://github.com/yairtes/The-8-Week-SQL-Challenge/blob/main/Case%20Study%20%233%20-%20Foodie-Fi/C.%20Challenge%20Payment%20Question.md).<br/>
To the 8-Week-Challenge site click [here](https://8weeksqlchallenge.com/case-study-1/)
