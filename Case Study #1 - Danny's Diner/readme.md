# 🍽️ Case Study #1 - Danny's Diner
This project explores various business problems for a fictional restaurant, **Danny’s Diner**, using SQL to derive insights from customer, sales, and menu data. The goal is to help Danny make data-driven decisions to improve customer retention and menu performance.
___
## 📚 Table of Content 
__⭕[Problem Statement](#Problem-Statement)__</br>
__⭕[Entity Relationship Diagram](#Entity-Relationship-Diagram)__</br>
__⭕[Tools Used](#tools-used)__</br>
__⭕[Challenge and Response](#Challenge-and-Response)__</br>
___
## 📍Problem Statement
 Danny wants to use the data to answer following Questions
  - Customers visiting Patterns.
  - Total Expenditure by Customers.
  - Customers favorite menu items.

Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.
He plans on using these insights to help him decide whether he should expand the existing customer loyalty program — additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.


The data set contains the following 3 tables which you may refer to the relationship diagram below to understand the connection.
 - sales
 - menu
 - members
___
## 📍Entity Relationship Diagram 
![Screenshot 2025-04-21 180820](https://github.com/user-attachments/assets/27083917-c916-4260-9316-90054ddd3543)

____
## 🛠️Tools Used
- 🐘 **PostgreSQL and DB-Fiddle** for SQL queries
- 📝 **Markdown** – for case documentation
___

 ## 📍Challenge and Response

__1. What is the total amount each customer spent at the restaurant?__

  ***Steps Taken***
- Used a **JOIN** on `sales` and `menu` tables via `product_id` to get item prices and compute total spend per customer.
  - `sales` table contains `customer_id` and `product_id`.
  - `menu` table contains `product_id` and `price`.
- Used **SUM** to aggregate the `Total Spent` by customers.
- User **Group and Order by** to aggregate the result and order them in ascending order.
  
***Solution***
```sql
select s.customer_id ,
sum(m.price) as Total_spent
from sales s
join menu m
on
s.product_id = m.product_id
group by s.customer_id
order by total_spent asc;
```
***Output***

|customer_id|	total_spent|
|------------|-----------|
|C|	36|
|B|	74|
|A|	76|

_____________
