# 🍕 Case Study #2 - Pizza Runner
This project analyzes operational and customer order data for a fictional pizza delivery startup, Pizza Runner.The goal is to help Pizza Runner improve delivery efficiency, optimize menu offerings, and enhance overall customer satisfaction.

_______

## 📚 Table of Content
__⭕[Problem Statement](#Problem-Statement)__</br>
__⭕[Entity Relationship Diagram](#Entity-Relationship-Diagram)__</br>
__⭕[Tools Used](#tools-used)__</br>
__⭕[Challenge and Response](#Challenge-and-Response)__
  - [Data Cleaning and Transformation](#Data-Cleaning-and-Transformation)
  - [A. Pizza Metrics](#Pizza-Metrics)
  - [B. Runner and Customer Experience](#Runner-and-Customer-Experience)
    
______

## ⁉️Problem Statement
Danny wants further assistance to clean his data and apply some basic calculations so he can better direct his runners and optimise Pizza Runner’s operations.

All datasets exist within the pizza_runner database schema are
- runners_orders
- customers_orders
- pizza_names
- pizza_recipes
- runners
- pizza_toppings

This case study is broken up  by area of focus including:
- Pizza Metrics
- Runner and Customer Experience
- Ingredient Optimisation
- Pricing and Ratings
- Bonus DML Challenges (DML = Data Manipulation Language)
But, Before we start writing our SQL queries, we want to investigate the data. We may need to handle some of the null values and adjust data types in the customer_orders and runner_orders tables.
_______
## 📍 Entity Relationship Diagram 
![image](https://github.com/user-attachments/assets/b683b7a7-4f07-4e2e-9f12-e75c6b1c285f)
______

____
## 🛠️Tools Used
- 🐘 **PostgreSQL and DB-Fiddle** for SQL queries
- 📝 **Markdown** – for case documentation
___

 ## 📍Challenge and Response
### Data Cleaning and Transformation
#### 🗃️ Table : Customer_orders
![image](https://github.com/user-attachments/assets/66a67f3d-5fb4-4fb6-a1bf-1aed39477f31)
- There is a need to clean the `exclusions` and `extra` column as they consist of **Null** values.
- Used **Temp** table `cust_clean` because it will be stored temporary in the database and we can access it easily.
- Used **Case** statement to replace the null values with a blank space.
```sql
create temp table cust_clean as
select 
order_id,
customer_id,
pizza_id,
case
	when exclusions is null or exclusions ='null' then ' '
    else exclusions
    end as exclusions,
case 
	when extras is null or extras = 'null' then ' '
    else  extras
    end as extras,
order_time
from customer_orders;
```
#### 🗃️ Table : runner_orders
![image](https://github.com/user-attachments/assets/fb0749f4-6dcc-45e4-a0ca-a1e443dead93)
- There is a need to clean :
    - `pickup_time` column as it contains **null** values.
    - `distance` column as it contains **null** values and **km** in the end which will create problem will executing queries.
    - `duration` column as it contains **null** values and **min** , **mins** , **minute** , **minutes** int the end.
    - `cancellation` column as it contains **null** values.
- Used **Temp** table `runner_clean` because it will be stored temporary in the database and we can access it easily.
- Used **Case** statement to replace the null values with a blank space and also for replacing unwanted values.
```sql
create temp table runner_clean as
	select
    order_id,
    runner_id,
    case 
     when pickup_time is null or pickup_time = 'null' then ' '
     else pickup_time
    end as pickup_time,
    case 
     when distance is null or distance ='null' then ' '
     when distance like '%km' then replace(distance , 'km', '')
     else distance
    end as distance,
    case 
    when duration is null or duration ='null' then ' '
    when duration like '%min'  then replace(duration ,'min','')
     when duration like '%mins'  then replace(duration ,'mins','')
     when duration like '%minute'  then replace(duration ,'minute','')
      when duration like '%minutes'  then replace(duration ,'minutes','')
    else duration
    end as duration,
    case
    when cancellation is null or cancellation ='null' then ' '
    else cancellation
    end as cancellation
from runner_orders;
  ```

__________

### ✔️ A. Pizza Metrics 

__1. How many pizzas were ordered?__

  ***Steps Taken***
- Used a **count** on `pizza_id` from **temp** table `cust_clean` to get the count of total no of pizza's ordered.
  
***Solution***
```sql
select 
	count(pizza_id) as No_of_orders
from
	cust_clean; 
```
***Output***


|no_of_orders|
|------|
|14|

_________

__2. How many unique customer orders were made?__

  ***Steps Taken***
- Used a **Distinct** and **count** on `order_id` from **temp** table `cust_clean` to get the Unique no of customers orders.
  
***Solution***
```sql
select
 count(distinct order_id) as unique_orders
 from
 cust_clean
```
***Output***


|unique_orders|
|---------|
|10|

_____________
__3. How many successful orders were delivered by each runner?__

  ***Steps Taken***
- Used  `runner_id` from **temp** table `runner_clean` to get the runners.
- Used **Count** on `duration` as delivered orders are counted as successful orders.
- used condition `where` to check if the distance travelled by runner is greater than 0
- used `group` and `order by` on `runner_id`
  
***Solution***
```sql
select 
	runner_id,
	count(duration) as No_of_orders
from 
	runner_clean
	where distance >'0'
	group by runner_id
	order by runner_id
```
***Output***



|runner_id|	no_of_orders|
|----------|--------------|
|1	|4|
|2|	3|
|3	|1|

_____________
__4. How many of each type of pizza was delivered?__

  ***Steps Taken***
- Used **Join** between `cust_clean` ,`runner_clean` and `pizza_names`.
  	- `cust_clean` contains the `pizza_id` .
  	- `pizza_names` contains `pizza_name`.
  	- `runner_clean` contains the `distance` for getting the sucessfull orders.
  
***Solution***
```sql

select c.pizza_id,
n.pizza_name,
count(r.duration) as delivered_pizzas
from
cust_clean c
join
pizza_names n
on
c.pizza_id = n.pizza_id
join
runner_clean r
on c.order_id = r.order_id
where distance > '0'
group by n.pizza_name , c.pizza_id
```
***Output***



|pizza_id|	pizza_name|	delivered_pizzas|
|----|-------|-------|
|1|	Meatlovers|	9|
|2|	Vegetarian|	3|


_____________

__5. How many Vegetarian and Meatlovers were ordered by each customer?__

  ***Steps Taken***
- Used **Join** between `cust_clean` and `pizza_names`.
  	- `cust_clean` contains  `pizza_id` and `customer_id`.
  	- `pizza_names` contains `pizza_name`.
- Grouped by `customer_id` and `pizza_name`.
- ordered by `customer_id`.
***Solution***
```sql

select c.customer_id,
n.pizza_name,
count(c.pizza_id) as No_of_orders
from
cust_clean c
join
pizza_names n
on 
c.pizza_id = n.pizza_id
group by c.customer_id, n.pizza_name
order by c.customer_id
```
***Output***
|customer_id	|pizza_name|	no_of_orders|
|------------|----------|------------|
|101	|Meatlovers	|2|
|101|	Vegetarian|	1|
|102	|Meatlovers	|2|
|102	|Vegetarian	|1|
|103	|Meatlovers	|3|
|103	|Vegetarian	|1|
|104	|Meatlovers	|3|
|105	|Vegetarian	|1|


________________________

__6. What was the maximum number of pizzas delivered in a single order?__

  ***Steps Taken***
- Used **Join** on `cust_clean` and `runner_clean`.
- Used `cust_clean` temp table as it contains `order_id` and `pizza_Id`.
- Used `runner_clean` temp table as it contains `distance`.
- Used `order by` on `total_orders` in desc order.
- Used `limit=1` to get the max orders.
***Solution***
```sql
select c.order_id,
count(c.pizza_id) as total_orders
from 
cust_clean c
join
runner_clean r
on c.order_id=r.order_id
where r.distance > '0'
group by c.order_id
order by total_orders desc
limit 1
```
***Output***

|order_id|	total_orders|
|------------|-----------|
|4	|3|

_____________________________________________________

__7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?__

  ***Steps Taken***
- Used **Join** on `cust_clean` and `runner_clean`.
- Used `cust_clean` temp table as it contains `exclusions` and `extra` which will dictate the changes.
- Used `runner_clean` temp table as it contains `distance` which will filters out the delivered pizzas .
- Used `order by` on `customer_id` .
***Solution***
```sql
select 
  c.customer_id,
  sum (
    case 
     when exclusions ='' and extras =''
     then 1 
     else 0 
    end ) as no_change,
  sum (
    case 
     when exclusions != ''  or extras != '' 
     then 1 
     else 0 
    end) as changes
from 
	cust_clean c
join 
	runner_clean r
	on c.order_id = r.order_id
where
	r.distance > '0'
group by 
	c.customer_id
order by 
	c.customer_id

```
***Output***

| customer_id | no_change | changes |
|-------------|-----------|---------|
| 101         | 2         | 0       |
| 102         | 3         | 0       |
| 103         | 0         | 3       |
| 104         | 1         | 2       |
| 105         | 0         | 1       |


_____________________________________________________
__8. How many pizzas were delivered that had both exclusions and extras?__

  ***Steps Taken***
- Used **Join** on `cust_clean` and `runner_clean`.
- Used `cust_clean` temp table as it contains `Pizza_id`.
- Used **count** for counting the pizza id's .
- Used **Where** to filter out the deliverd pizzas and to find the pizza with both `exclusions` and `extras` .
***Solution***
```sql
select 
	count(c.pizza_id) as Pizza
from
	cust_clean c
join
	runner_clean r
	on c.order_id = r.order_id
where 
	(r.distance > '0') 
    and (c.exclusions != '' and c.extras !='') ;


```
***Output***


|pizza|
|-----|
|1|
_____________________________________________________

__9. What was the total volume of pizzas ordered for each hour of the day?__

  ***Steps Taken***
- Used **extract** as hour to get the hour from  `order_time`.
- Used **count** for counting the pizza id's .
***Solution***
```sql

select
extract(hour from order_time) as Order_hours,
count(pizza_id) as pizza_count
from cust_clean
group by order_hours
order by order_hours

```
***Output***


| order_hours | pizza_count |
|-------------|-------------|
| 11          | 1           |
| 13          | 3           |
| 18          | 3           |
| 19          | 1           |
| 21          | 3           |
| 23          | 3           |

_____________________________________________________
__10. What was the volume of orders for each day of the week?__

  ***Steps Taken***
- Used `cust_clean` temp table as it contains `order_id`.
- Used **extract** as isodow to get the day of the week from  `order_time`.
- - Used **count** for counting the pizza id's .
***Solution***
```sql

select
extract(isodow from order_time) as day_number,
count(order_id) as order_count
from 
cust_clean
group by day_number
order By day_number

​




```
***Output***


| day_number | order_count |
|------------|-------------|
| 3          | 5           |
| 4          | 3           |
| 5          | 1           |
| 6          | 5           |

_____________________________________________________


## 🏃🏼‍♀️B. Runner and Customer Experience
