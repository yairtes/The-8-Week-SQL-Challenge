# :shopping_cart:	 Case Study #4: Data Mart :shopping_cart:		

## Solution A. Data Cleansing Steps

### 1. In a single query, perform the following operations and generate a new table in the `data_mart` schema named `clean_weekly_sales`:

- Convert `the week_date` to a DATE format

- Add a `week_number` as the second column for each week_date value, for example any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2 etc

- Add a `month_number` with the calendar month for each `week_date` value as the 3rd column

- Add a `calendar_year` column as the 4th column containing either 2018, 2019 or 2020 values

- Add a new column called `age_band` after the original segment column using the following mapping on the number inside the segment value

segment	| age_band
--|--
1	|Young Adults
2	|Middle Aged
3 or 4|	Retirees

- Add a new `demographic` column using the following mapping for the first letter in the segment values:

segment|	demographic
--|--
C	|Couples
F	|Families

- Ensure all `null` string values with an "unknown" string value in the original `segment` column as well as the new `age_band` and `demographic` columns

- Generate a new `avg_transaction` column as the `sales` value divided by `transactions` rounded to 2 decimal places for each record

````sql
DROP TABLE IF EXISTS clean_weekly_sales
SELECT
		CONVERT(DATE, week_date,3) week_date,
		DATEPART(WEEK, CONVERT(DATE, week_date,3)) week_number,
		DATEPART(MONTH, CONVERT(DATE, week_date,3)) month_number,
		DATEPART(YEAR, CONVERT(DATE, week_date,3)) year,
		region,
		platform,
		CASE
			WHEN segment LIKE '%1%' THEN 'young adults'
			WHEN segment LIKE '%2%' THEN 'Middle Aged'
			WHEN segment LIKE '%3%' OR segment LIKE '%4%' THEN 'Retirees'
			ELSE 'unknown'
			END age_band,
		CASE 
			WHEN segment LIKE '%c%' THEN 'Couples'
			WHEN segment LIKE '%f%' THEN 'families'
			ELSE 'unknown'
			END demographic,
		transactions,
		sales,
		ROUND(CAST(sales AS FLOAT)/CAST(transactions AS FLOAT), 2) avg_transaction 
INTO clean_weekly_sales
FROM weekly_sales
````
NOTE: Not all ouptut is displayed

#### Answer:

week_date | week_number | month_number | year | region | platform | age_band | demographic | transactions | sales | avg_transaction
--|--|--|--|--|--|--|--|--|--|--
2020-08-31	|36	|8	|2020	|ASIA	|Retail	|Retirees	|Couples	|120631	|3656163	|30.31
2020-08-31	|36	|8	|2020	|ASIA	|Retail	|young adults	|families	|31574	|996575	|31.56
2020-08-31	|36	|8	|2020	|USA	|Retail	|unknown	|unknown	|529151	|16509610	|31.2
2020-08-31	|36	|8	|2020	|EUROPE	|Retail	|young adults	|Couples	|4517	|141942	|31.42
2020-08-31	|36	|8	|2020	|AFRICA	|Retail	|Middle Aged	|Couples	|58046	|1758388	|30.29
2020-08-31	|36	|8	|2020	|CANADA	|Shopify	|Middle Aged	|families	|1336	|243878	|182.54
2020-08-31	|36	|8	|2020	|AFRICA	|Shopify	|Retirees	|families	|2514	|519502	|206.64
2020-08-31	|36	|8	|2020	|ASIA	|Shopify	|young adults	|families	|2158	|371417	|172.11
2020-08-31	|36	|8	|2020	|AFRICA	|Shopify	|Middle Aged	|families	|318	|49557	|155.84
2020-08-31	|36	|8	|2020	|AFRICA	|Retail	|Retirees	|Couples	|111032	|3888162	|35.02

***

<p align="center">
  :nerd_face:	:nerd_face:	:nerd_face:	
</p>

### Links :link:

For any Questions, comments or better solutions, feel free to contact me on my [LinkedIn account](https://www.linkedin.com/in/yair-teshuva/).<br/>
To the next solution, B.Data Exploration click [here](https://github.com/yairtes/The-8-Week-SQL-Challenge/blob/main/Case%20Study%20%235%20-%20Data%20Mart/B.%20Data%20Exploration.md)<br/>
To the 8-Week-Challenge site click [here](https://8weeksqlchallenge.com/case-study-1/)
