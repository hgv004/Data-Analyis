# Case Study #2 - Pizza Runner
![image](https://github.com/hgv004/Project/assets/105195779/f37c6fd8-a623-40a5-a877-54ddf49eebb7)

## Introduction
- Did you know that over 115 million kilograms of pizza is consumed daily worldwide??? (Well according to Wikipedia anyway…)

- Danny was scrolling through his Instagram feed when something really caught his eye - “80s Retro Styling and Pizza Is The Future!”

- Danny was sold on the idea, but he knew that pizza alone was not going to help him get seed funding to expand his new Pizza Empire - so he had one more genius idea to combine with it - he was going to Uberize it - and so Pizza Runner was launched!

- Danny started by recruiting “runners” to deliver fresh pizza from Pizza Runner Headquarters (otherwise known as Danny’s house) and also maxed out his credit card to pay freelance developers to build a mobile app to accept orders from customers.

## Available Data
- Because Danny had a few years of experience as a data scientist - he was very aware that data collection was going to be critical for his business’ growth.

- He has prepared for us an entity relationship diagram of his database design but requires further assistance to clean his data and apply some basic calculations so he can better direct his runners and optimise Pizza Runner’s operations.

## Database Creation
```sql
CREATE SCHEMA IF NOT EXISTS pizza_runner;
USE pizza_runner;

DROP TABLE IF EXISTS runners;
CREATE TABLE runners (
  runner_id INT,
  registration_date DATE
);

INSERT INTO runners (runner_id, registration_date)
VALUES
  (1, '2021-01-01'),
  (2, '2021-01-03'),
  (3, '2021-01-08'),
  (4, '2021-01-15');

DROP TABLE IF EXISTS customer_orders;
CREATE TABLE customer_orders (
  order_id INT,
  customer_id INT,
  pizza_id INT,
  exclusions VARCHAR(4),
  extras VARCHAR(4),
  order_time TIMESTAMP
);
INSERT INTO customer_orders (order_id, customer_id, pizza_id, exclusions, extras, order_time)
VALUES
  (1, 101, 1, '', '', '2020-01-01 18:05:02'),
  (2, 101, 1, '', '', '2020-01-01 19:00:52'),
  (3, 102, 1, '', '', '2020-01-02 23:51:23'),
  (3, 102, 2, '', NULL, '2020-01-02 23:51:23'),
  (4, 103, 1, '4', '', '2020-01-04 13:23:46'),
  (4, 103, 1, '4', '', '2020-01-04 13:23:46'),
  (4, 103, 2, '4', '', '2020-01-04 13:23:46'),
  (5, 104, 1, NULL, '1', '2020-01-08 21:00:29'),
  (6, 101, 2, NULL, NULL, '2020-01-08 21:03:13'),
  (7, 105, 2, NULL, '1', '2020-01-08 21:20:29'),
  (8, 102, 1, NULL, NULL, '2020-01-09 23:54:33'),
  (9, 103, 1, '4', '1, 5', '2020-01-10 11:22:59'),
  (10, 104, 1, NULL, NULL, '2020-01-11 18:34:49'),
  (10, 104, 1, '2, 6', '1, 4', '2020-01-11 18:34:49');

DROP TABLE IF EXISTS runner_orders;
CREATE TABLE runner_orders (
  order_id INT,
  runner_id INT,
  pickup_time VARCHAR(19),
  distance VARCHAR(7),
  duration VARCHAR(10),
  cancellation VARCHAR(23)
);
INSERT INTO runner_orders (order_id, runner_id, pickup_time, distance, duration, cancellation)
VALUES
  (1, 1, '2020-01-01 18:15:34', '20km', '32 minutes', ''),
  (2, 1, '2020-01-01 19:10:54', '20km', '27 minutes', ''),
  (3, 1, '2020-01-03 00:12:37', '13.4km', '20 mins', NULL),
  (4, 2, '2020-01-04 13:53:03', '23.4', '40', NULL),
  (5, 3, '2020-01-08 21:10:57', '10', '15', NULL),
  (6, 3, NULL, NULL, NULL, 'Restaurant Cancellation'),
  (7, 2, '2020-01-08 21:30:45', '25km', '25mins', NULL),
  (8, 2, '2020-01-10 00:15:02', '23.4 km', '15 minute', NULL),
  (9, 2, NULL, NULL, NULL, 'Customer Cancellation'),
  (10, 1, '2020-01-11 18:50:20', '10km', '10minutes', NULL);

DROP TABLE IF EXISTS pizza_names;
CREATE TABLE pizza_names (
  pizza_id INT,
  pizza_name TEXT
);
INSERT INTO pizza_names (pizza_id, pizza_name)
VALUES
  (1, 'Meatlovers'),
  (2, 'Vegetarian');

DROP TABLE IF EXISTS pizza_recipes;
CREATE TABLE pizza_recipes (
  pizza_id INT,
  toppings TEXT
);
INSERT INTO pizza_recipes (pizza_id, toppings)
VALUES
  (1, '1, 2, 3, 4, 5, 6, 8, 10'),
  (2, '4, 6, 7, 9, 11, 12');

DROP TABLE IF EXISTS pizza_toppings;
CREATE TABLE pizza_toppings (
  topping_id INT,
  topping_name TEXT
);
INSERT INTO pizza_toppings (topping_id, topping_name)
VALUES
  (1, 'Bacon'),
  (2, 'BBQ Sauce'),
  (3, 'Beef'),
  (4, 'Cheese'),
  (5, 'Chicken'),
  (6, 'Mushrooms'),
  (7, 'Onions'),
  (8, 'Pepperoni'),
  (9, 'Peppers'),
  (10, 'Salami'),
  (11, 'Tomatoes'),
  (12, 'Tomato Sauce');
```

# Solution
## Pizza Metrics
### 1. How many pizzas were ordered?
```sql
select count(*)from customer_orders co ;
```
### 2. How many unique customer orders were made?
```sql
select count(distinct order_id) as unique_orders
from customer_orders co ;
```

### 3. How many successful orders were delivered by each runner?
```sql
CREATE TABLE runner_orders1 AS
SELECT order_id, runner_id, pickup_time, distance, duration, IF(cancellation = '', NULL, cancellation) AS cancellation
FROM pizza_runner.runner_orders;


select runner_id , count(order_id)
from runner_orders1 ro
where isnull(cancellation )
group by runner_id;
```
### 4. How many of each type of pizza was delivered?
```sql
select co.pizza_id , count(co.order_id) as pizza_delivered
from customer_orders co 
left join runner_orders1 ro 
on ro.order_id = co.order_id 
where isnull(ro.cancellation )
group by co.pizza_id;
```

### 5. How many Vegetarian and Meatlovers were ordered by each customer?
```sql
select co.customer_id, pn.pizza_name , count(order_id) as total_pizza
from customer_orders co 
left join pizza_names pn 
on pn.pizza_id = co.pizza_id 
group by co.customer_id, pn.pizza_name
order by co.customer_id, pn.pizza_name;
```
### 6. What was the maximum number of pizzas delivered in a single order?
```sql
select co.order_id , count(co.customer_id) as pizza_delivered
from customer_orders co 
left join runner_orders1 ro 
on ro.order_id = co.order_id 
where isnull(ro.cancellation) 
group by co.order_id
order by count(co.customer_id) desc
limit 1;
```
### 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
```sql
CREATE TABLE customer_orders1 AS
select 	order_id, customer_id, pizza_id, 
		IF(exclusions = '', NULL, exclusions) AS exclusions, 
		IF(extras = '', NULL, extras) AS extras, 
		order_time
FROM pizza_runner.customer_orders;

select 	co.customer_id, 
		count(co.order_id) as total_pizza,
		sum(if(isnull(co.exclusions) and isnull(co.extras),1,0)) as no_change,
		sum(if(not isnull(co.exclusions) or not isnull(co.extras),1,0)) as atleast_one_change
from customer_orders1 co 
left join runner_orders1 ro 
on ro.order_id = co.order_id 
where isnull(ro.cancellation) 
group by co.customer_id;

-- for Testing 

select 	co.customer_id, 
		co.extras ,
		co.exclusions 
from customer_orders1 co 
left join runner_orders1 ro 
on ro.order_id = co.order_id 
where isnull(ro.cancellation)
order by co.customer_id ;
```
### 8. How many pizzas were delivered that had both exclusions and extras?
```sql
select 	co.customer_id, 
		sum(if(not isnull(co.exclusions) and not isnull(co.extras),1,0)) as atleast_one_change
from customer_orders1 co 
left join runner_orders1 ro 
on ro.order_id = co.order_id 
where isnull(ro.cancellation) 
group by co.customer_id
having atleast_one_change >0;
```
### 9. What was the total volume of pizzas ordered for each hour of the day?
```sql
select 	date(order_time) as dt,
		hour (order_time) as hr,
		count(order_id) as pizza_volume
FROM pizza_runner.customer_orders1
group by dt,hr
order by dt,hr;
```
### 10. What was the volume of orders for each day of the week?
```sql
select 	dayname(order_time) as Days,
		count(order_id) as pizza_volume
FROM pizza_runner.customer_orders1
group by Days
order by Days;
```
## B. Runner and Customer Experience

### Preparing date Table
```sql
use pizza_runner;

create table date_table
	(WITH RECURSIVE 
	all_date (_date) AS
		(
		SELECT "2020-01-01"
		union all
		SELECT date_add(_date, INTERVAL 1 DAY) from all_date where _date < "2020-12-31"
		)
	SELECT max(_date) FROM all_date);
```
### 1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01) [Look]
```sql
select week(registration_date) as _week, count(runner_id) as sign_up 
from runners r 
group by _week;
```
### 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
```sql
with order_time as (
	select order_id, max(order_time) as ot
	from customer_orders1
	group by order_id) 
select ro.runner_id, avg(minute(timediff(od.ot, ro.pickup_time))) as avg_mins
from runner_orders1 ro
inner join  order_time od
on ro.order_id = od.order_id
group by ro.runner_id;
```
### 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?
```sql
with order_time as (
	select order_id, max(order_time) as ot, count(order_id) as pizzas
	from customer_orders1
	group by order_id) 
select ro.order_id , sum(od.pizzas) as total_pizzas, sum(minute(timediff(od.ot, ro.pickup_time))) as total_mins 
from runner_orders1 ro
inner join  order_time od
on ro.order_id = od.order_id
where ro.pickup_time is not null
group by ro.order_id;
```
### 4. What was the average distance travelled for each customer?
```sql
select 	co.customer_id ,
		avg(if(substring(distance,3,1) =".",left(distance,4),left (distance,2))) as dist
from runner_orders1 ro
inner join  customer_orders1 co 
on ro.order_id = co.order_id
where ro.pickup_time is not null
group by co.customer_id;
```
### 5. What was the difference between the longest and shortest delivery times for all orders?
```sql
select  (max(left(duration,2))- min(left(duration,2))) as diff
from runner_orders1 ro;
```
### 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?
```sql
select r.runner_id, avg(speed) as avg_speed
from (
	select 	runner_id ,
			if(substring(distance,3,1) =".",left(distance,4),left (distance,2)) 
			/
			(left(duration,2) / 60 ) as speed
	from runner_orders1 ro 
	where duration is not null) as r
group by r.runner_id;
```
### 7. What is the successful delivery percentage for each runner?
```sql
select runner_id , (sum(if(duration is not null,1,0)) / count(order_id))*100 as succ_delivery
from runner_orders1 ro
group by runner_id;
```
## **C. Ingredient Optimisation**
### Transforming the pizza_recipes table
```sql
-- Create a temporary view for demonstration
CREATE OR REPLACE TEMP VIEW temp_pizza_recipes AS
SELECT pizza_id, split(toppings, ', ') AS toppings_array
FROM pizza_runner.pizza_recipes;

-- Query to unnest the toppings array using TRANSFORM and LATERAL VIEW
create or replace table pizza_runner.pizza_recipes as
SELECT
  pizza_id,
  CAST(topping AS INTEGER) AS topping_id -- Assuming topping_id should be an integer
FROM temp_pizza_recipes
LATERAL VIEW EXPLODE(toppings_array) AS topping;
```
### 1. What are the standard ingredients for each pizza?
```sql
SELECT
  pn.pizza_name,
  array_join(collect_set(pt.topping_name), ', ')  as toppings
FROM
  pizza_runner.pizza_recipes AS pr
LEFT JOIN pizza_runner.pizza_names AS pn 
ON pr.pizza_id = pn.pizza_id
left join pizza_runner.pizza_toppings pt
on pt.topping_id = pr.topping_id
group by 1;
```

### Updating the Customer_orders table
```sql
UPDATE pizza_runner.customer_orders
SET extras = NULL
WHERE extras IN ('', 'null');

UPDATE pizza_runner.customer_orders
SET exclusions = NULL
WHERE exclusions IN ('', 'null');
```


;
### Transforming Customer_orders table
```sql
CREATE OR REPLACE TEMP VIEW temp_cust_orders AS
SELECT
  order_id,
  customer_id,
  pizza_id,
  order_time,
  SPLIT(extras, ', ') AS extra_array,
  SPLIT(exclusions, ', ') AS exclusions_array
FROM pizza_runner.customer_orders;


-- Assuming topping_id should be an integer, hence casting extras and exclusions as INTEGER
CREATE OR REPLACE TABLE pizza_runner.customer_orders AS
SELECT
  order_id,
  customer_id,
  pizza_id,
  order_time,
  CAST(extra_id AS INTEGER) AS extra_id,
  CAST(exclusion_id AS INTEGER) AS exclusion_id
FROM temp_cust_orders
LATERAL VIEW EXPLODE(extra_array) temp_cust_orders AS extra_id
LATERAL VIEW EXPLODE(exclusions_array) temp_cust_orders AS exclusion_id;
```

### 2. What was the most commonly added extra?
```sql
select pt.topping_name, count(pt.topping_name) no_of_orders
 from pizza_runner.customer_orders co 
inner join pizza_runner.pizza_toppings pt
on pt.topping_id = co.extra_id
group by 1 
order by 2 desc;
```
### 3. What was the most common exclusion?
```sql
select pt.topping_name, count(pt.topping_name) no_of_orders
 from pizza_runner.customer_orders co 
inner join pizza_runner.pizza_toppings pt
on pt.topping_id = co.exclusion_id
group by 1 
order by 2 desc;
```
### 4. Generate an order item for each record in the customers_orders table in the format of one of the following:
- `Meat Lovers`
- `Meat Lovers - Exclude Beef`
- `Meat Lovers - Extra Bacon`
- `Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers`
```sql
select
  co.order_id,
  pn.pizza_name,
  -- array_join(collect_set(pt1.topping_name), ', ') exclusion_list,
  -- array_join(collect_set(pt2.topping_name), ', ') extra_list,
  -- concat(
  --   pn.pizza_name,
  --   " - Exlclude ",
  --   array_join(collect_set(pt1.topping_name), ', '),
  --   ", - Extra ",
  --   array_join(collect_set(pt2.topping_name), ', ')
  -- ),
  case
    when array_join(collect_set(pt1.topping_name), ', ') = ''
    and array_join(collect_set(pt2.topping_name), ', ') = '' then pn.pizza_name
    when array_join(collect_set(pt1.topping_name), ', ') = '' then concat(
      pn.pizza_name,
      " - Extra ",
      array_join(collect_set(pt2.topping_name), ', ')
    )
    when
    array_join(collect_set(pt2.topping_name), ', ') = '' then concat(
      pn.pizza_name,
      " - Exlclude ",
      array_join(collect_set(pt1.topping_name), ', ')
    )
    else concat(
      pn.pizza_name,
      " - Exlclude ",
      array_join(collect_set(pt1.topping_name), ', '),
      " - Extra ",
      array_join(collect_set(pt2.topping_name), ', ')
    )
  end as final_order
from
  pizza_runner.customer_orders co
  left join pizza_runner.pizza_toppings pt1 on pt1.topping_id = co.exclusion_id
  left join pizza_runner.pizza_toppings pt2 on pt2.topping_id = co.extra_id
  left join pizza_runner.pizza_names pn on pn.pizza_id = co.pizza_id
group by
  1,
  2
order by
  1;
```
### 5. Generate an alphabetically ordered comma-separated ingredient list for each pizza order from the customer_orders table and add a `2x` in front of any relevant ingredients
    - For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"
```sql
with cte1 as (
  SELECT
  co.order_id,
  co.pizza_id,
  co.extra_id,
  pr.topping_id,
  pt.topping_name,
  CASE
    WHEN co.extra_id = pr.topping_id THEN '2x-'
    ELSE null
  END AS alias
FROM
  pizza_runner.customer_orders co
LEFT JOIN pizza_runner.pizza_recipes pr ON pr.pizza_id = co.pizza_id
LEFT JOIN pizza_runner.pizza_toppings pt ON pt.topping_id = pr.topping_id
group by 1, 2, 3, 4, pt.topping_name
ORDER BY
  co.order_id,
  co.pizza_id,
  pt.topping_name),
cte2 as (
select c.order_id, 
  pn.pizza_name, 
  -- c.topping_name, 
  -- c.alias, 
  case when c.alias is null then c.topping_name
  else concat(c.alias, c.topping_name) end as combined
from cte1 c
left join pizza_runner.pizza_names pn on pn.pizza_id = c.pizza_id
qualify row_number() over(partition by c.order_id, c.pizza_id, c.topping_name order by c.order_id, c.pizza_id, c.topping_name, c.alias desc) = 1
order by 1, 2, c.topping_name)
select order_id, concat(pizza_name, "-", array_join(collect_set(combined), ', ')) as  final_order_with_extra
from cte2
group by 1, pizza_name
order by 1 ;

```
### 6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?
```sql
select
  pt1.topping_name,
  sum(case
    when pt.topping_id = co.extra_id then 2
    else 1
  end) as orders
from
  pizza_runner.customer_orders co
  left join pizza_runner.pizza_recipes pt on co.pizza_id = pt.pizza_id
  left join pizza_runner.pizza_toppings pt1 on pt.topping_id = pt1.topping_id
where
  co.exclusion_id is null
  or pt.topping_id <> co.exclusion_id
group by 1
order by 2 desc
```

## **D. Pricing and Ratings**

### 1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?
```sql
select 	ro.runner_id, 
		sum(case when pn.pizza_name = "Meatlovers" then 12 when pn.pizza_name = "Vegetarian" then 10 else 0 end) total_money
from customer_orders1 co 
inner join runner_orders1 ro 
on ro.order_id = co.order_id 
inner join pizza_names pn 
on pn.pizza_id = co.pizza_id 
where ro.duration is not null
group by ro.runner_id;
```
### 2. What if there was an additional $1 charge for any pizza extras?
    - Add cheese is $1 extra
```sql
select 	ro.runner_id  , 
		sum(case when extras = 1 then 2 when extras is not null then 1 else 0 end
		+
		case when c.pizza_id = 1 then 12 when c.pizza_id = 2 then 10 else 0 end) as total
from extras e 
left join co1 c 
on c.transaction_id = e.transaction_id 
left join runner_orders1 ro 
on ro.order_id = c.order_id 
where ro.duration is not null
group by ro.runner_id;
```
### 3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.
```sql
drop table rating_table;
create table rating_table (
	select order_id, if(duration is not null, FLOOR(1 + RAND() * 5),null) rating  
	from runner_orders1 ro) ;
```
### 4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
    - `customer_id`
    - `order_id`
    - `runner_id`
    - `rating`
    - `order_time`
    - `pickup_time`
    - Time between order and pickup
    - Delivery duration
    - Average speed
    - Total number of pizzas
```sql
select 	co.customer_id, 
		co.order_id, 
		ro.runner_id, 
		rt.rating,
		co.order_time ,
		ro.pickup_time ,
		minute (timediff(ro.pickup_time, co.order_time)) diff,
		ro.duration ,
		if(substring(ro.distance,3,1) =".",left(ro.distance,4),left (ro.distance,2)) 
			/
			(left(ro.duration,2) / 60 ) as speed_kmh			
from customer_orders1 co 
inner join runner_orders1 ro 
on ro.order_id = co.order_id 
left join rating_table rt 
on rt.order_id = co.order_id
where ro.cancellation is null;
```
### 5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?
```sql
create table pizza_with_rate (
select *, if(pizza_id = 1, 12,10) as rate
from pizza_names pn );

select 	sum(pwr.rate) as rev,
		sum(if(substring(ro.distance,3,1) =".",left(ro.distance,4),left (ro.distance,2)))*0.3 as extra_cost,
		sum(pwr.rate) - (sum(if(substring(ro.distance,3,1) =".",left(ro.distance,4),left (ro.distance,2)))*0.3) as profit
from customer_orders1 co 
left join pizza_with_rate pwr 
on pwr.pizza_id = co.pizza_id 
left join runner_orders1 ro 
on ro.order_id = co.order_id 
where ro.cancellation is null;
```
