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

__2. How many days has each customer visited the restaurant?__

  ***Steps Taken***
- Used `customer_id` and `order_date` from table `sales`.
- Used **Count** to count the `Days Visited` by customers.
- Used **Distinct** to count the unique visits as there were multiple orders.
- User **Group and Order by** to aggregate the result and order them in ascending order.
  
***Solution***
```sql
select customer_id ,
count( distinct order_date) as Days_Visited
from sales
group by customer_id
order by customer_id;
```
***Output***

|customer_id|	days_visited|
|-----|------|
|A|	4|
|B|	6|
|C	|2|

_____________

__3. What was the first item from the menu purchased by each customer?__

  ***Steps Taken***
- Used a **JOIN** between `sales` table and `menu` table.
  - `sales` table contains `customer_id` and `order_date`.
  - `menu` table contains `product_name`.
- Used **Dense_Rank()** window function in the select command - so if a customer ordered multiple items on the same first date, all are shown.( As only date is given not timestamp)
  - Partition the data by customer_id so that the ranking is done for each individual customer.
  - Order the results by order_date to ensure that the earliest order gets the rank of 1.
  - This will allow customers who ordered multiple items on the same day to receive the same rank.
- In the WHERE clause, filter the results to only include rows where rank = 1, which represents the first order for each customer.
- Retrieved the customer_id and product_name for each first purchase.
  
***Solution***
```sql
select 
summary.customer_id,
summary.product_name
from(
  select 
  s.customer_id,
  m.product_name,
  dense_rank() over
  (partition by s.customer_id order by s.order_date) As Rnk
  from sales s 
  join menu m
  on 
  s.product_id=m.product_id)
  as summary
where Rnk=1;
```
***Output***

|customer_id	|product_name|
|-----------|------------|
|A|	curry|
|A|	sushi|
|B|	curry|
|C	|ramen|
|C	|ramen|

__________________________________
__4. What is the most purchased item on the menu and how many times was it purchased by all customers?__

  ***Steps Taken***
- Used a **JOIN** between `sales` table and `menu` table.
  - `sales` table contains `customer_id`.
  - `menu` table contains `product_name`.
- Used **Count** to count the total occurrence of the item.
- Grouped using the `product_name` and ordered by `no_of_purchase` in desc order.
- Used `Limit=1` to get only the top item.
  
***Solution***
```sql
select
 m.product_name,
 count(s.product_id) As No_of_purchase
from
 menu m
join
 sales s
on m.product_id=s.product_id
group by
 m.product_name
order by
  no_of_purchase desc 
 limit 1
```
***Output***

|product_name|	no_of_purchase|
|-----|-----|
|ramen	|8|


__________________________________

__5. Which item was the most popular for each customer?__

  ***Steps Taken***
- Used a **JOIN** between `sales` table and `menu` table.
  - `sales` table contains `customer_id` and  `product_id`.
  - `menu` table contains `product_name`.
- Used **Count** for getting the No of purchase of particular order.
- Used **Row_number()** window function , partitioned by `customer_id` and order by `purchase count ` in **desc** order.
- Used `rn=1` to get top ranked product per customer.
  
***Solution***
```sql
select 
	ranked.customer_id,
	ranked.product_name
from
	(select 
     s.customer_id,
	 m.product_name,
	 count(s.product_id) as purchase_count,
	 row_number() over 
     (partition by s.customer_id order by count(s.product_id)desc ) as rn
from
	sales s
join 
     menu m
on s.product_id=m.product_id
group by
     s.customer_id , m.product_name
order by 
     s.customer_id) as ranked
where 
	rn=1
```
***Output***

|customer_id	|product_name|
|-----|------|
|A	|ramen|
|B|	ramen|
|C	|ramen|

__________________________________
__6. Which item was purchased first by the customer after they became a member?__

  ***Steps Taken***
- Used a **JOIN** between `sales` table , `menu` table and `members ` table.
  - `sales` table contains `customer_id` ,  `order_date` and `product_id` .
  - `menu` table contains `product_name`.
  - `members` table contains `join_date`.
- Used **Row_number()** window function , partitioned by `customer_id` and order by ` order_date ` in **asc** order.
- Used `where order_date > join_date ` to filter out the orders that were placed after the `join_date`.
- Used `rn=1` to get first order purchased by the members.
  
***Solution***
```sql
select
	rm.customer_id,
    rm.order_date,
    rm.product_name
from (select 
	mem.customer_id,
	s.order_date ,
    m.product_name,
  row_number() over
  (partition by mem.customer_id order by s.order_date ) as rn
from
	members mem
join
	sales s
on 
	mem.customer_id=s.customer_id
join
	menu m
on
	s.product_id=m.product_id
where 
	s.order_date > mem.join_date
) as rm
    where rn =1
    

```
***Output***

| customer_id |order_date | product_name |
|--------|---------|--------|
|A|	2021-01-10|	ramen|
|B	|2021-01-11	|sushi|

____________________

__7. Which item was purchased just before the customer became a member?__

  ***Steps Taken***
- Used a **JOIN** between `sales` table , `menu` table and `members ` table.
  - `sales` table contains `customer_id` ,  `order_date` and `product_id` .
  - `menu` table contains `product_name`.
  - `members` table contains `join_date`.
- Used **Row_number()** window function , partitioned by `customer_id` and order by ` order_date ` in **desc** order as it gave the recent date .
- Used `where order_date < join_date ` to filter out the orders that were placed after the `join_date`.
- Used `rn=1` to get first order purchased by the members.
  
***Solution***
```sql
select
	rm.customer_id,
    rm.product_name,
    rm.order_date
from
	(select 
	mem.customer_id,
    s.order_date,
    m.product_name,
    row_number() over
    (partition by mem.customer_id order by s.order_date desc) As rn
from
	members mem
join
	sales s 
on
	mem.customer_id = s.customer_id
join
	menu m
on 
	s.product_id = m.product_id
where 
	s.order_date < mem.join_date) as rm
    where rn=1;


```
***Output***

|customer_id|	product_name|	order_date|
|----------------|-----------|-----------|
|A|	sushi|	2021-01-01|
|B|	sushi	|2021-01-04|

_______________________________________________

__8. What is the total items and amount spent for each member before they became a member?__

  ***Steps Taken***
- Used a **JOIN** between `sales` table , `menu` table and `members ` table.
  - `sales` table contains `customer_id` ,  `order_date` and `product_id` .
  - `menu` table contains `price`.
  - `members` table contains `join_date`.
- Used **sum** on `price` from `menu` table to calculate the total sales amount.
- Used **Count** on `product_id` from `sales` table to count the number of items purchased.
- Used `where order_date < join_date ` to filter the orders before the joining of members.
- Grouped and Ordered By `Customer_id`.
  
***Solution***
```sql
select 
	mem.customer_id,
    sum(m.price) as Sales,
    count(s.product_id) as Items
from
	members mem
join 
	sales s
on
	mem.customer_id=s.customer_id
join
	menu m
on
	s.product_id = m.product_id
where
	s.order_date < mem.join_date
group by
	mem.customer_id
order by 
	mem.customer_id
```
***Output***

|customer_id|	sales|	items|
|----|-------|-----|
|A|	25	|2|
|B|	40	|3|

_______________________________________________
__9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?__

  ***Steps Taken***
- Used a **JOIN** between `sales` table and `menu` table.
  - `sales` table contains `customer_id`  and `product_id` .
  - `menu` table contains `price` and `product_name`.
- Used **sum** to calculate the total points.
- Used **case** for checking condition that is - if sushi then points is 2x otherwise 10.
- Grouped and Ordered By `Customer_id`.
  
***Solution***
```sql
select
	s.customer_id,
    sum(
      case
      	when m.product_name='sushi' then m.price*20
      	else m.price*10
      	end) as Total_Points
from
 	sales s
join
	menu m
on	
	s.product_id=m.product_id
group by
	s.customer_id
order by		
	s.customer_id

```
***Output***

|customer_id	|total_points|
|----------|----------|
|A|	860|
|B	|940|
|C|	360|

_______________________________________________
__10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January ?__

  ***Steps Taken***
- Used a **JOIN** between `sales` table , `menu` table and `members` table .
  - `sales` table contains `customer_id`  and `order_date` .
  - `menu` table contains `price` and `product_name`.
  - `members` table contains `join_date`.
- Used **sum** to calculate the total points.
- Used **case** for checking condition that is
  - if sushi than 2x.
  - if the `order_date` is between the week from `join_date` than 2x.
  - else 10 points.
- Grouped and Ordered By `Customer_id`.
- Used `where order_date  < 31 jan ` for filtering the orders before 31 jan
  
***Solution***
```sql
select 
	s.customer_id,
	sum(case 
        	when m.product_name = 'sushi' then m.price*20
        	when s.order_date between mem.join_date and mem.join_date + interval '7 day' 
        	then m.price*20
        else
        	m.price*10
        end) as Points
from
	sales s
 join
 	menu m
 on
 	s.product_id=m.product_id
join
	members mem
on
	s.customer_id=mem.customer_id
where
	s.order_date < '2021-01-31'
group by
	s.customer_id
order by
	s.customer_id

```
***Output***

|customer_id|	points|
|------------|-----------|
|A|	1370|
|B	|940|


_______________________________________________

