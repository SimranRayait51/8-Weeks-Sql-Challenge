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
    end as extras
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
	where duration >'0'
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
  	- `runner_clean` contains the `duration` for getting the sucessfull orders.
  
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
where duration > '0'
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
- Used `runner_clean` temp table as it contains `duration`.
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
where r.duration > '0'
group by c.order_id
order by total_orders desc
limit 1
```
***Output***

|order_id|	total_orders|
|------------|-----------|
|4	|3|

_____________________________________________________


