# **Case Study #6 - Clique Bait**
![image](https://github.com/hgv004/Data-Analyis/assets/105195779/38c7131c-2852-4d9f-8dbc-4c8d9950d52e)

## Introduction
- Clique Bait is not like your regular online seafood store - the founder and CEO Danny, was also a part of a digital data analytics team and wanted to expand his knowledge into the seafood industry!

- In this case study - you are required to support Danny’s vision and analyse his dataset and come up with creative solutions to calculate funnel fallout rates for the Clique Bait online store.

# **Case Study Questions**
## 1. Enterprise Relationship Diagram
```sql
CREATE TABLE clique_bait.event_identifier (
  "event_type" INTEGER,
  "event_name" VARCHAR(13)
);

CREATE TABLE clique_bait.campaign_identifier (
  "campaign_id" INTEGER,
  "products" VARCHAR(3),
  "campaign_name" VARCHAR(33),
  "start_date" TIMESTAMP,
  "end_date" TIMESTAMP
);

CREATE TABLE clique_bait.page_hierarchy (
  "page_id" INTEGER,
  "page_name" VARCHAR(14),
  "product_category" VARCHAR(9),
  "product_id" INTEGER
);

CREATE TABLE clique_bait.users (
  "user_id" INTEGER,
  "cookie_id" VARCHAR(6),
  "start_date" TIMESTAMP
);

CREATE TABLE clique_bait.events (
  "visit_id" VARCHAR(6),
  "cookie_id" VARCHAR(6),
  "page_id" INTEGER,
  "event_type" INTEGER,
  "sequence_number" INTEGER,
  "event_time" TIMESTAMP
);
```

## **2. Digital Analysis**

#### 1. How many users are there?
```sql
select count( distinct user_id)
from users;
-- How many cookies does each user have on average?
with cte as (
    select user_id, count( cookie_id) as ct
    from users
    group by user_id
    order by count( cookie_id) desc)
select round(avg(ct), 0) as avg_cookies
from cte;
```
#### 2. How many cookies does each user have on average?
```sql
SELECT 
  EXTRACT(MONTH FROM event_time) as month, 
  COUNT(DISTINCT visit_id) AS unique_visit_count
FROM events
GROUP BY EXTRACT(MONTH FROM event_time);
-- What is the number of events for each event type?
SELECT 
  event_type, 
  COUNT(*) AS event_count
FROM events
GROUP BY event_type
ORDER BY event_type;
```
#### 3. What is the unique number of visits by all users per month?
```sql
SELECT 
  round((count( distinct(iff(ei.event_name = 'Purchase', e.visit_id, null)))
  / 
  COUNT(distinct visit_id))*100,2) AS unique_visits_pct
FROM events e
left join event_identifier ei
on e.event_type = ei.event_type;
```
#### 4. What is the number of events for each event type?
```sql
SELECT 
  event_type, 
  COUNT(*) AS event_count
FROM events
GROUP BY event_type
ORDER BY event_type;
```
#### 5. What is the percentage of visits which have a purchase event?
```sql
SELECT 
  round((count( distinct(iff(ei.event_name = 'Purchase', e.visit_id, null)))
  / 
  COUNT(distinct visit_id))*100,2) AS unique_visits_pct
FROM events e
left join event_identifier ei
on e.event_type = ei.event_type;
```
#### 6. What is the percentage of visits which view the checkout page but do not have a purchase event?
```sql

```
#### 7. What are the top 3 pages by number of views?
```sql
SELECT 
  ph."page_name",
  count(distinct e.visit_id) as views
FROM events e
left join event_identifier ei
on e.event_type = ei.event_type
left join page_hierarchy ph
on ph."page_id" = e.page_id
group by ph."page_name"
order by 2 desc
limit 3;
```
#### 8. What is the number of views and cart adds for each product category?
```sql
SELECT 
  ph."product_category",
  count(iff(ei.event_name='Page View', e.visit_id, null)) as total_views,
  count(iff(ei.event_name='Add to Cart', e.visit_id, null)) as cart_ads
FROM events e
left join event_identifier ei
on e.event_type = ei.event_type
left join page_hierarchy ph
on ph."page_id" = e.page_id
group by ph."product_category"
order by 2 desc;
```
#### 9. What are the top 3 products by purchases?
```sql
select 
  ph."product_category",
  count(e.event_type)
FROM events e
left join event_identifier ei
on e.event_type = ei.event_type
left join page_hierarchy ph
on ph."page_id" = e.page_id
group by ph."product_category";
```

