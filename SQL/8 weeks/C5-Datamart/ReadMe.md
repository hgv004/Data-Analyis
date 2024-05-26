# **Case Study #5 - Data Mart**
![image](https://github.com/hgv004/Data-Analyis/assets/105195779/c1d47ea0-617f-4828-848b-64108a6fd152)

## Introduction
- Data Mart is Danny’s latest venture and after running international operations for his online supermarket that specialises in fresh produce - Danny is asking for your support to analyse his sales performance.

- In June 2020 - large scale supply changes were made at Data Mart. All Data Mart products now use sustainable packaging methods in every single step from the farm all the way to the customer.

- Danny needs your help to quantify the impact of this change on the sales performance for Data Mart and it’s separate business areas.

# **Case Study Questions**
## 1. Data Cleansing Steps
#### 1. Convert the week_date to a DATE format
```sql
ALTER TABLE weekly_sales
ADD COLUMN wk_date DATE;

UPDATE weekly_sales
SET wk_date = STR_TO_DATE(week_date, '%d/%m/%y');
```

#### 2. Add a week_number as the second column for each week_date value, for example any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2 etc
#### Add a month_number with the calendar month for each week_date value as the 3rd column
#### Add a calendar_year column as the 4th column containing either 2018, 2019 or 2020 values
#### segment	age_band 1	Young Adults   2	Middle Aged 3 or 4	Retirees
#### Add a new demographic column using the following mapping for the first letter in the segment values: segment	demographic C	Couples F	Families
#### Ensure all null string values with an "unknown" string value in the original segment column as well as the new age_band and demographic columns
#### Generate a new avg_transaction column as the sales value divided by transactions rounded to 2 decimal places for each record
```sql
create table clean_sales as 
select 	*, 
		week(wk_date) as week_number , 
		month(wk_date) as month_number ,
		year(wk_date) as calendar_year,
		case 
		WHEN RIGHT(segment,1) = '1' THEN 'Young Adults'
	    WHEN RIGHT(segment,1) = '2' THEN 'Middle Aged'
	    WHEN RIGHT(segment,1) in ('3','4') THEN 'Retirees'
	    ELSE 'unknown' END AS age_band,
	    CASE 
	    WHEN LEFT(segment,1) = 'C' THEN 'Couples'
	    WHEN LEFT(segment,1) = 'F' THEN 'Families'
	    ELSE 'unknown' END AS demographic,
	    ROUND((sales/transactions),2) AS avg_transaction
from weekly_sales;
```
## 2. **Data Exploration**
#### 1. What day of the week is used for each `week_date` value?
```sql
select distinct format(dates, 'dddd') as day_of_week
from data_mart.weekly_sales_clean
where transactions is not null;
```
#### 2. What range of week numbers are missing from the dataset?
```sql
WITH Numbers AS (
    SELECT 1 AS v1
    UNION ALL
    SELECT v1 + 1
    FROM Numbers
    WHERE v1 < 52
), temp_table as (
	select distinct wk_no as day_of_week
	from data_mart.weekly_sales_clean
	where transactions is not null)
SELECT v1
FROM Numbers
left join temp_table tt
on tt.day_of_week = Numbers.v1
where tt.day_of_week is null
OPTION (MAXRECURSION 0);
```
#### 3. How many total transactions were there for each year in the dataset?
```sql
select year(dates) as yr, sum(transactions) as total_transactions
from data_mart.weekly_sales_clean
where transactions is not null
group by year(dates)
order by year(dates);
```
#### 4. What is the total sales for each region for each month?
```sql
select month(dates) as _month, region, sum(CAST(sales AS BIGINT)) as total_sales
from data_mart.weekly_sales_clean
where sales is not null
group by month(dates), region
order by month(dates), region;
```
#### 5. What is the total count of transactions for each platform
```sql
select PLATFORM, count(transactions) as total_transactions
from data_mart.weekly_sales_clean
where transactions is not null
group by PLATFORM;
```
#### 6. What is the percentage of sales for Retail vs Shopify for each month?
```sql
select	month(dates) as _month, 
		(sum(iif(PLATFORM ='Shopify', CAST(sales AS decimal(10,2)),0)) /
		sum(CAST(sales AS bigint)) * 100)  as Shopify_sales_pct,  
		(sum(iif(PLATFORM ='Retail', CAST(sales AS decimal(10,2)),0)) /
		sum(CAST(sales AS bigint)) * 100)  as Retail_sales_pct
from data_mart.weekly_sales_clean
where transactions is not null
group by month(dates)
order by month(dates);
```
#### 7. What is the percentage of sales by demographic for each year in the dataset?
```sql
select	year(dates) as yr, 
		(sum(iif(demographic ='Couples', CAST(sales AS decimal(10,2)),0)) /
		sum(CAST(sales AS bigint)) * 100)  as Couple_sales_pct,  
		(sum(iif(demographic ='Families', CAST(sales AS decimal(10,2)),0)) /
		sum(CAST(sales AS bigint)) * 100)  as Family_sales_pct,
		(sum(iif(demographic ='Unknown', CAST(sales AS decimal(10,2)),0)) /
		sum(CAST(sales AS bigint)) * 100)  as Family_sales_pct
from data_mart.weekly_sales_clean
where transactions is not null
group by year(dates) 
order by year(dates) ;
```
#### 8. Which `age_band` and `demographic` values contribute the most to Retail sales?
```sql
select	top 1 age_band, 
		demographic
from data_mart.weekly_sales_clean
where transactions is not null and PLATFORM ='Retail' and demographic <> 'Unknown'
group by age_band, demographic
order by sum(CAST(sales AS bigint)) desc;
```
#### 9. Can we use the `avg_transaction` column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?
```sql
select	year(dates) as yr,
		sum(iif(PLATFORM ='Shopify', cast(sales as decimal),0)) / 
			sum(iif(PLATFORM ='Shopify', transactions,0)) as shopify_tra_size,
		sum(iif(PLATFORM ='Retail', cast(sales as decimal),0)) / 
			sum(iif(PLATFORM ='Retail', transactions,0)) as shopify_tra_size
from data_mart.weekly_sales_clean
where transactions is not null
group by year(dates);
```
