# 🍕 Case Study #2 - Pizza Runner
This project analyzes operational and customer order data for a fictional pizza delivery startup, Pizza Runner.The goal is to help Pizza Runner improve delivery efficiency, optimize menu offerings, and enhance overall customer satisfaction.

_______

## 📚 Table of Content
__⭕[Problem Statement](#Problem-Statement)__</br>
__⭕[Entity Relationship Diagram](#Entity-Relationship-Diagram)__</br>
__⭕[Tools Used](#tools-used)__</br>
__⭕[Challenge and Response](#Challenge-and-Response)__
  - [Data Cleaning and Transformation](#Data-Cleaning-and-Transformation)
    
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
