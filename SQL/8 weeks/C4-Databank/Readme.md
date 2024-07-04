# Case Study #4 - Data Bank
![image](https://github.com/hgv004/Project/assets/105195779/87bf12b7-531f-4f4f-a3f5-1dbddb228ab4)


## Introduction
- There is a new innovation in the financial industry called Neo-Banks: new aged digital only banks without physical branches.

- Danny thought that there should be some sort of intersection between these new age banks, cryptocurrency and the data world…so he decides to launch a new initiative - Data Bank!

- Data Bank runs just like any other digital bank - but it isn’t only for banking activities, they also have the world’s most secure distributed data storage platform!

- Customers are allocated cloud data storage limits which are directly linked to how much money they have in their accounts. There are a few interesting caveats that go with this business model, and this is where the Data Bank team need your help!

- The management team at Data Bank want to increase their total customer base - but also need some help tracking just how much data storage their customers will need.

- This case study is all about calculating metrics, growth and helping the business analyse their data in a smart way to better forecast and plan for their future developments!

## **Case Study Questions**

The following case study questions include some general data exploration analysis for the nodes and transactions before diving right into the core business questions and finishes with a challenging final request!

## **A. Customer Nodes Exploration**

### 1. How many unique nodes are there on the Data Bank system?
```sql
select count(distinct node_id ) as unique_nodes
from customer_nodes cn ;
```
![image](https://github.com/hgv004/Data-Analyis/assets/105195779/f66de034-ac85-4ead-99b9-ddb013dd6269)

### 2. What is the number of nodes per region?
```sql
select r.region_name , count(node_id) as node_counts
from customer_nodes cn
join regions r 
on r.region_id = cn.region_id 
group by r.region_name ;
```
![image](https://github.com/hgv004/Data-Analyis/assets/105195779/ebfcfe62-363b-4a8b-a67a-9e2bc3056a23)

### 3. How many customers are allocated to each region?
```sql
select r.region_name , count(distinct customer_id) as customers
from customer_nodes cn
join regions r 
on r.region_id = cn.region_id 
group by r.region_name ;
```
![image](https://github.com/hgv004/Data-Analyis/assets/105195779/d9566672-2c42-43bb-9c55-ffc5f2decd6d)

### 4. How many days on average are customers reallocated to a different node?
```sql
select avg(datediff(end_date, start_date)) as avg_days
from customer_nodes cn 
where year(end_date) !=9999;
```
![image](https://github.com/hgv004/Data-Analyis/assets/105195779/58bf0dab-385b-4276-a8ec-42a0d3976b36)

### 5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?
```sql
WITH date_diff AS
(
	SELECT cn.customer_id,
	       cn.region_id,
	       r.region_name,
	       DATEDIFF(DAY, start_date, end_date) AS reallocation_days
	FROM customer_nodes cn
	INNER JOIN regions r
	ON cn.region_id = r.region_id
	WHERE end_date != '9999-12-31'
)
SELECT DISTINCT region_id,
	        region_name,
	        PERCENTILE_CONT(0.5) WITHIN GROUP(ORDER BY reallocation_days) OVER(PARTITION BY region_name) AS median,
	        PERCENTILE_CONT(0.8) WITHIN GROUP(ORDER BY reallocation_days) OVER(PARTITION BY region_name) AS percentile_80,
	        PERCENTILE_CONT(0.95) WITHIN GROUP(ORDER BY reallocation_days) OVER(PARTITION BY region_name) AS percentile_95
FROM date_diff
ORDER BY region_name;
```
![image](https://github.com/hgv004/Data-Analyis/assets/105195779/9f3e6bb3-0d1e-4f92-8df1-17fa8037c8e7)

## **B. Customer Transactions**
- In Order to eliminate repeatative Query I made one Customer balance table which tells us about the customer balance between 2 dates "from_date" and "to_date"
- Assumed the last date of customer transaction is the highest date value from the table
### Customer Balance table
```sql
create or replace table cust_bal as (
SELECT
    txn_date as from_date,
    ifnull(lead(txn_date) over(PARTITION BY customer_id order by txn_date) -1,last_day(txn_date))  as to_date,
    customer_id,
    sum(
      if(txn_type = 'deposit', txn_amount, 0 - txn_amount)
    ) over(
      partition by customer_id
      order by
        txn_date rows between unbounded preceding
        and current row
    ) as running_bal
  FROM
    data_bank.customer_transactions
  order by
    customer_id, txn_date)
```
![image](https://github.com/hgv004/Data-Analyis/assets/105195779/150acc72-1e58-4229-81e5-1b21ad121912)

### 1. What is the unique count and total amount for each transaction type?
```sql
select txn_type , sum(txn_amount) as total_amt, count(customer_id) as unique_counts
from customer_transactions ct 
group by txn_type;
```
![image](https://github.com/hgv004/Data-Analyis/assets/105195779/59f09312-14fa-4588-b477-0a4c870ec5fe)
- Deposits are higher compared to Purchases and withdrawals.
### 2. What is the average total historical deposit counts and amounts for all customers?
```sql
with cte as (
	select customer_id , count(txn_type) as deposits, sum(txn_amount) as total_amt
	from customer_transactions ct 
	where txn_type  = 'deposit'
	group by customer_id
	order by customer_id)
select avg(cte.deposits) avg_deposit_count, avg(cte.total_amt) avg_deposit_amount
from cte;
```
![image](https://github.com/hgv004/Data-Analyis/assets/105195779/c3ed6d26-3032-4422-8b0e-34f2b0fda1c3)
- On and average 6 deposits are made per  customers.
- Avg deposit amount is 2718.3 per customer.

### 3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?
```sql
with cte as (
	select 	DATE_FORMAT(txn_date , "%M- %Y") as my,
			customer_id ,
			count(if(txn_type = 'deposit', txn_type,null)) as deposit,
			count(if(txn_type = 'purchase', txn_type,null)) as purchase,
			count(if(txn_type = 'withdrawal', txn_type,null)) as withdrawal
	from customer_transactions ct 
	group by my, customer_id
	having deposit > 1 and purchase + withdrawal !=0
	order by my, customer_id)
select cte.my, count(distinct cte.customer_id) as customer_counts
from cte
group by cte.my
order by cte.my;
```
![image](https://github.com/hgv004/Data-Analyis/assets/105195779/21eabf2d-9958-4888-81cf-981d21da42b2)
- March-2020 seems more active month in terms of deposit and withdrawals in same month by customer.

### 4. What is the closing balance for each customer at the end of the month?
```sql
with month_end_dates as (
  select distinct last_day(txn_date) as month_dates
FROM
    data_bank.customer_transactions
order by 1)
select
  m.month_dates as month_end_date,
  c.customer_id,
  c.running_bal as month_end_balance
from
  cust_bal c
left join month_end_dates m
on m.month_dates between c.from_date and c.to_date
where month_dates is not null
order by 2, 1;
```
![image](https://github.com/hgv004/Data-Analyis/assets/105195779/c1530ab4-96b0-4f55-9c1a-ea9bb59d6e4d)
- Image shows only top 6 records for each customer's month end balance.

### 5. What is the percentage of customers who increase their closing balance by more than 5%?
```sql

```
## **C. Data Allocation Challenge**

- To test out a few different hypotheses - the Data Bank team wants to run an experiment where different groups of customers would be allocated data using 3 different options:
- In Order to reduce redundant query I used Temp view which generates the date table which contains the dates ranging from min date of the table and max date.
### Date-Range Table 
```sql
CREATE or replace TEMPORARY VIEW date_range AS (
  SELECT explode(sequence((SELECT MIN(txn_date) FROM data_bank.customer_transactions), 
                          (SELECT MAX(txn_date) FROM data_bank.customer_transactions), 
                          interval 1 day)) AS date
);
```
![image](https://github.com/hgv004/Data-Analyis/assets/105195779/70ad5abb-1cc8-4438-b506-6363cf16e8a0)

### Option 1: data is allocated based off the amount of money at the end of the previous month
- I extracted last day of each month and joined with the cust_balance table with joining condition of the ending date falling between the from date to to_date values from the cust_balance table.
```sql
with month_end_dates as (
  select distinct last_day(txn_date) as month_dates
FROM
    data_bank.customer_transactions
order by 1)
select
  m.month_dates as prev_month_end_date,
  m.month_dates+1 as curr_month_date, sum(c.running_bal) as data_required
from
  cust_bal c
left join month_end_dates m
on m.month_dates between c.from_date and c.to_date
where month_dates is not null
group by 1,2
order by 1;
```
![image](https://github.com/hgv004/Data-Analyis/assets/105195779/d508f48f-31ef-4a5c-9ce9-c9126dc32ed9)

### Option 2: data is allocated on the average amount of money kept in the account in the previous 30 days
 ```sql
 with past_30days_avg_table as(
  SELECT
    d.date,
    c.customer_id,
    c.running_bal,
    round(
      avg(c.running_bal) over(
        PARTITION BY c.customer_id
        order by
          d.date rows between 30 preceding
          and current row
      ),
      1
    ) past_30days_avg
  FROM
    date_range d
    left join cust_bal c on d.date BETWEEN c.from_date
    and c.to_date
)
select
  date(date_trunc('month', p.date)) as month,
  round(sum(p.past_30days_avg),0) as data_required
from
  past_30days_avg_table p
group by
  1
order by
  1;
  ```
![image](https://github.com/hgv004/Data-Analyis/assets/105195779/20d54ea0-b863-4c59-b8db-f1d5140d8e05)

### Option 3: data is updated real-time
```sql
with daily_bal_table as(
  SELECT
    d.date,
    c.customer_id,
    c.running_bal
  FROM
    date_range d
    left join cust_bal c on d.date BETWEEN c.from_date
    and c.to_date
)
select
  date(date_trunc('month', d.date)) as month,
  round(sum(d.running_bal),0) as data_required
from
  daily_bal_table d
group by
  1
order by
  1;
```
![image](https://github.com/hgv004/Data-Analyis/assets/105195779/c8b3badd-9e7d-475a-915a-deb75d1e1140)

For this multi-part challenge question - you have been requested to generate the following data elements to help the Data Bank team estimate how much data will need to be provisioned for each option:

### 1. Running customer balance column that includes the impact each transaction
```sql
SELECT 	customer_id ,
		txn_date ,
		txn_type ,
		if(txn_type = 'deposit', txn_amount , 0-txn_amount) as txn,
		sum(if(txn_type = 'deposit', txn_amount , 0-txn_amount)) 
			over(partition by customer_id order by txn_date rows between unbounded preceding  and current row) as running_bal
FROM customer_transactions
order by customer_id , txn_date ;
```
### 2. Customer balance at the end of each month
```sql
with cte as (
	SELECT 	customer_id ,
			date_format(txn_date ,"%b-%y") as my,
			last_day(txn_date) as ld,
			sum(if(txn_type = 'deposit', txn_amount , 0-txn_amount)) as total_txn
	FROM customer_transactions
	group by customer_id , my, ld
	order by customer_id , ld)
select 	cte.customer_id , 
		cte.ld,
		sum(cte.total_txn) over(partition by customer_id order by cte.ld) as month_balance
from cte;
```
### 3. Minimum, average and maximum values of the running balance for each customer
```sql
with running_txn as (
  SELECT 	customer_id,
		sum(if(txn_type = 'deposit', txn_amount , 0-txn_amount)) 
			over(partition by customer_id order by txn_date rows between unbounded preceding  and current row) as running_bal
  FROM data_bank.customer_transactions
  order by customer_id)
select  customer_id, 
        round(avg(running_txn.running_bal),1) as avg_bal, 
        min(running_txn.running_bal) as min_bal, 
        max(running_txn.running_bal) as max_bal
from running_txn
group by 1;
```

### 4. Using all of the data available - how much data would have been required for each option on a monthly basis?
```sql

```
## **D. Extra Challenge**

- Data Bank wants to try another option which is a bit more difficult to implement - they want to calculate data growth using an interest calculation, just like in a traditional savings account you might have with a bank.

### If the annual interest rate is set at 6% and the Data Bank team wants to reward its customers by increasing their data allocation based off the interest calculated on a daily basis at the end of each day, how much data would be required for this option on a monthly basis?
```sql
with daily_bal_table as(
  SELECT
    d.date,
    c.customer_id,
    c.running_bal,
    if(c.running_bal > 0, (0.06/365)*running_bal, 0) as interest_earn
  FROM
    date_range d
    left join cust_bal c on d.date BETWEEN c.from_date
    and c.to_date
)
select
  date(date_trunc('month', d.date)) as month,
  round(sum(d.running_bal + d.interest_earn),0) as data_required
from
  daily_bal_table d
group by
  1
order by
  1;
```



### Special notes : 
Data Bank wants an initial calculation which does not allow for compounding interest, however they may also be interested in a daily compounding interest calculation.
```sql

```
## **Extension Request**

The Data Bank team wants you to use the outputs generated from the above sections to create a quick Powerpoint presentation which will be used as marketing materials for both external investors who might want to buy Data Bank shares and new prospective customers who might want to bank with Data Bank.

#### 1. Using the outputs generated from the customer node questions, generate a few headline insights which Data Bank might use to market its world-leading security features to potential investors and customers.
```sql

```
#### 2. With the transaction analysis - prepare a 1 page presentation slide which contains all the relevant information about the various options for the data provisioning so the Data Bank management team can make an informed decision.
```sql

```
