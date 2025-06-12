# 🍽️ Case Study #3 - Foodie-Fi

Welcome to my SQL case study solution for **Case Study #3: Foodie-Fi**, part of the [8 Week SQL Challenge](https://8weeksqlchallenge.com/) by [Danny Ma](https://twitter.com/datawithdanny). This challenge simulates a real-world data problem related to a subscription-based meal service platform.

---
## 📚 Table of Contents

- ⭕ Problem Statement  
- ⭕ Entity Relationship Diagram  
- ⭕ Tools Used  
- ⭕ Data Exploration & Cleaning  
- ⭕ Challenge Areas  
  - A. Subscription Analytics  
  - B. Customer Retention & Revenue  
  - C. Lifetime Value & Recommendations  

---

## ⁉️ Problem Statement

Foodie-Fi offers streaming meal content through various subscription plans. Each new user begins with a free trial, and can transition between plans: basic, pro, and lifetime.

**Danny**, the founder, wants insights into customer behavior and plan performance, such as:

- How many users convert after free trials?
- Which plans generate the most revenue?
- How long do users stay on each plan?
- What strategies can be developed for churn reduction and upselling?

---

## 🧱 Database Schema

All datasets exist within the `foodie_fi` schema and include the following tables:

- `plans` – Subscription plan details (name, price)
- `subscriptions` – Customer-level plan transitions and start dates

**Entity Relationship Diagram (ERD):**
![image](https://github.com/user-attachments/assets/e4bdd32b-171d-41a1-a3a2-b4b89874aabc)
____
## 🛠️Tools Used
- 🐘 **PostgreSQL and DB-Fiddle** for SQL queries
- 📝 **Markdown** – for case documentation
___

 ## 📍Challenge and Response
 ### A. Customer Journey
Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.

Try to keep it as short as possible - you may also want to run some sort of join to make your explanations a bit easier!

***Solution***
``` sql
SELECT 
s.customer_id,
STRING_AGG(
  p.plan_name, ' → ' ORDER BY s.start_date) as Journey
from
plans p
join
subscriptions s
on
p.plan_id =s.plan_id
where
customer_id in(1,2,11,13,15,16,18,19)
group by s.customer_id
```
**Output**
| Customer ID | Journey                                 |
|-------------|------------------------------------------|
| 1           | trial → basic monthly                    |
| 2           | trial → pro annual                       |
| 11          | trial → churn                            |
| 13          | trial → basic monthly → pro monthly      |
| 15          | trial → pro monthly → churn              |
| 16          | trial → basic monthly → pro annual       |
| 18          | trial → pro monthly                      |
| 19          | trial → pro monthly → pro annual         |

**Observations**
| Customer ID | Onboarding Journey Description                                      |
|-------------|---------------------------------------------------------------------|
| 1           | Started with a trial and moved to a basic monthly plan.             |
| 2           | Began with a trial and upgraded directly to a pro annual plan.      |
| 11          | Used the trial and churned without subscribing to a paid plan.      |
| 13          | Moved from trial to basic monthly, then upgraded to pro monthly.    |
| 15          | Started with a trial, subscribed to pro monthly, and then churned.  |
| 16          | Went from trial to basic monthly, then upgraded to pro annual.      |
| 18          | Transitioned from trial straight to a pro monthly plan.             |
| 19          | Switched from trial to pro monthly, then upgraded to pro annual.    |

________
 ### B. Data Analysis Questions
 __1. How many customers has Foodie-Fi ever had?__
 **Steps Taken**
 - Used `Count` and `Distinct` to count all unique customer.
**Solution**
``sql
select
count(distinct customer_id) as Customers
from
subscriptions;
``
**Output**
| customers |
|-----------|
| 1000      |

_________
__2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value.__

  ***Steps Taken***
- Used ``Extract`` to get the `month_id ` from the `start_date` column of `subscriptions` table.
- Used ``To_char`` to get the  `Month_name`.
- Used `Count` to count all the customer where `plan_name` is trial.
***Solution***
```sql
select
extract(month from s.start_date)as month_id,
to_char(s.start_date , 'Month') as month_name,
count(s.customer_id) as customers
from
subscriptions s
join 
plans p
on
p.plan_id=s.plan_id
where
plan_name='trial'
group by month_name, month_id 
order by month_id
```
***Output***
| month_id | month_name | customers |
|----------|------------|-----------|
| 1        | January    | 88        |
| 2        | February   | 68        |
| 3        | March      | 94        |
| 4        | April      | 81        |
| 5        | May        | 88        |
| 6        | June       | 79        |
| 7        | July       | 89        |
| 8        | August     | 88        |
| 9        | September  | 87        |
| 10       | October    | 79        |
| 11       | November   | 75        |
| 12       | December   | 84        |

 - March has the highest number of customers.
_______
__3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name.__

  ***Steps Taken***
- Used ``Extract`` to get the `Year ` from the `start_date` and applied condition.
***Solution***
```sql
select
p.plan_id,
p.plan_name,
count(s.customer_id) as events
from
subscriptions s
join
plans p
on
s.plan_id=p.plan_id
where extract(YEAR from start_date) > 2020
group by p.plan_id, p.plan_name
order by p.plan_id;
```
***Output***
| plan_id | plan_name      | events |
|---------|----------------|--------|
| 1       | basic monthly  | 8      |
| 2       | pro monthly    | 60     |
| 3       | pro annual     | 63     |
| 4       | churn          | 71     |

_______
__3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name.__

  ***Steps Taken***
- Used ``Extract`` to get the `Year ` from the `start_date` and applied condition.
***Solution***
```sql
select
p.plan_id,
p.plan_name,
count(s.customer_id) as events
from
subscriptions s
join
plans p
on
s.plan_id=p.plan_id
where extract(YEAR from start_date) > 2020
group by p.plan_id, p.plan_name
order by p.plan_id;
```
***Output***
| plan_id | plan_name      | events |
|---------|----------------|--------|
| 1       | basic monthly  | 8      |
| 2       | pro monthly    | 60     |
| 3       | pro annual     | 63     |
| 4       | churn          | 71     |

_______

__3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name.__

  ***Steps Taken***
- Used ``Extract`` to get the `Year ` from the `start_date` and applied condition.
***Solution***
```sql
select
p.plan_id,
p.plan_name,
count(s.customer_id) as events
from
subscriptions s
join
plans p
on
s.plan_id=p.plan_id
where extract(YEAR from start_date) > 2020
group by p.plan_id, p.plan_name
order by p.plan_id;
```
***Output***
| plan_id | plan_name      | events |
|---------|----------------|--------|
| 1       | basic monthly  | 8      |
| 2       | pro monthly    | 60     |
| 3       | pro annual     | 63     |
| 4       | churn          | 71     |

_______

__3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name.__

  ***Steps Taken***
- Used ``Extract`` to get the `Year ` from the `start_date` and applied condition.
***Solution***
```sql
select
p.plan_id,
p.plan_name,
count(s.customer_id) as events
from
subscriptions s
join
plans p
on
s.plan_id=p.plan_id
where extract(YEAR from start_date) > 2020
group by p.plan_id, p.plan_name
order by p.plan_id;
```
***Output***
| plan_id | plan_name      | events |
|---------|----------------|--------|
| 1       | basic monthly  | 8      |
| 2       | pro monthly    | 60     |
| 3       | pro annual     | 63     |
| 4       | churn          | 71     |

_______

__3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name.__

  ***Steps Taken***
- Used ``Extract`` to get the `Year ` from the `start_date` and applied condition.
***Solution***
```sql
select
p.plan_id,
p.plan_name,
count(s.customer_id) as events
from
subscriptions s
join
plans p
on
s.plan_id=p.plan_id
where extract(YEAR from start_date) > 2020
group by p.plan_id, p.plan_name
order by p.plan_id;
```
***Output***
| plan_id | plan_name      | events |
|---------|----------------|--------|
| 1       | basic monthly  | 8      |
| 2       | pro monthly    | 60     |
| 3       | pro annual     | 63     |
| 4       | churn          | 71     |

_______

__4.What is the customer count and percentage of customers who have churned rounded to 1 decimal place?__

  ***Steps Taken***
- Used ``Extract`` to get the `Year ` from the `start_date` and applied condition.
***Solution***
```sql
select
p.plan_id,
p.plan_name,
count(s.customer_id) as events
from
subscriptions s
join
plans p
on
s.plan_id=p.plan_id
where extract(YEAR from start_date) > 2020
group by p.plan_id, p.plan_name
order by p.plan_id;
```
***Output***
| plan_id | plan_name      | events |
|---------|----------------|--------|
| 1       | basic monthly  | 8      |
| 2       | pro monthly    | 60     |
| 3       | pro annual     | 63     |
| 4       | churn          | 71     |
