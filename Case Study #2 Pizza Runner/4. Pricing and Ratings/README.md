# [8WeeksSQLChallange](https://github.com/sweety21-coder/8WeekSQLChallange)
# 🍕 Case Study #2 - Pizza Metrics
<p align="center">
<img src= "https://github.com/sweety21-coder/8WeekSQLChallange/blob/main/IMG/Pizza%20Runner.png?raw=true" width=50%,height=50%>

 ## 📋 Case Study's Questions

  1.If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has 
  Pizza Runner made so far if there are no delivery fees?

 2.What if there was an additional $1 charge for any pizza extras?
  Add cheese is $1 extra.

 3.The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner,
  how would you design an additional table for this new dataset - generate a schema for this new table and insert 
  Your own data for ratings for each successful customer order between 1 to 5.

 4.Using your newly generated table - can you join all of the information together to form a table which has the following information
  for successful deliveries?
customer_id,
order_id,
runner_id,
rating,
order_time,
pickup_time,
Time between order and pickup,
Delivery duration,
Average speed,
Total number of pizzas

5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - 
   how much money does Pizza Runner have left over after these deliveries?

### E. Bonus Questions
                
If Danny wants to expand his range of pizzas - how would this impact the existing data design? Write an INSERT statement to 
demonstrate what would happen if a new Supreme pizza with all the toppings was added to the Pizza Runner menu?

## 🔑 Solutions
### **Q1.If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has** 
         **Pizza Runner made so far if there are no delivery fees?**
```Query
select
sum(CASE WHEN c.pizza_id= 1 THEN 12 ELSE 10 END) as Money_earned
from #updated_customer_orders c
join #updated_runner_orders r
on c.order_id=r.order_id
where distance <> 0;
```
**Result**
|MoneyEarned|
|-----------|
| 138       |

**Money Earned by Pizza Runner is 138$**

### **Q2.What if there was an additional $1 charge for any pizza extras? Add cheese is $1 extra.**
```Query
select
SUM(CASE WHEN pizza_id= 1 THEN 12 ELSE 10 END)+
SUM(CASE WHEN extras is not null or extras <> '' THEN 1 ELSE 0 END) as Total_earned
from #updated_customer_orders c
join #updated_runner_orders r
on c.order_id=r.order_id
where distance <> 0;
```
**Result**
|Total_earned |
|-------------|
| 150         |

Total Money earned by including additional charge for extras and cheese is 150$


### **Q3.The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner,how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.**
```Query
create table Ratings(
Customer_id Integer,
Order_id Integer,
Runner_id integer,
Rating integer)
Insert into Ratings values
(101,1,1,5),
(101,2,1,5),
(102,3,1,4),
(103,4,2,4),
(104,5,3,3),
(105,7,2,5),
(102,8,2,4),
(104,10,1,4);

select * from Ratings;
```
**Result**
|Customer_id|Order_id|Runner_id|Rating|
|-----------|--------|---------|------|
|101        |1       |1        |5     |
|101        |2       |1        |5     |
|102        |3       |1        |4     |
|103        |4       |2        |4     |
|104        |5       |3        |3     |
|105        |7       |2        |5     |
|102        |8       |2        |4     |
|104        |10      |1        |4     |

### **Q4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?**
customer_id,
order_id,
runner_id,
rating,
order_time,
pickup_time,
Time between order and pickup,
Delivery duration,
Average speed,
Total number of pizzas
 
```Query
select c.customer_id,c.order_id,ro.runner_id,Rating,order_time,
pickup_time ,DATEDIFF(MINUTE,order_time,pickup_time)as time_between_order_and_pickup,duration,
ROUND(avg((convert(float,distance) / convert(float,duration))*60),2) AS average_speed,COUNT(*)as Pizza_count
from #updated_customer_orders c
join #updated_runner_orders ro
on c.order_id=ro.order_id
join Ratings r
on c.order_id= r.Order_id
where distance <> 0
group by c.customer_id,c.order_id,ro.runner_id,Rating,order_time,
pickup_time,duration
order by order_id;
```
**Result**
 |Order_id |Runner_id |Rating |  order_time               |pickup_time              |time_between_order_and_pickup |duration |average_speed |Pizza_count |
 |---------|----------|-------|---------------------------|-------------------------|------------------------------|---------|--------------|------------|
 |1        |1         |5      |2020-01-01 18:05:02.000    |2020-01-01 18:15:34.000  |10                            |32       |37.5          |1           |
 |2        |1         |5      |2020-01-01 19:00:52.000    |2020-01-01 19:10:54.000  |10                            |27       |44.44         |1           |
 |3        |1         |4      |2020-01-02 23:51:23.000    |2020-01-03 00:12:37.000  |21                            |20       |40.2          |2           |
 |4        |2         |4      |2020-01-04 13:23:46.000    |2020-01-04 13:53:03.000  |30                            |40       |35.1          |3           |
 |5        |3         |3      |2020-01-08 21:00:29.000    |2020-01-08 21:10:57.000  |10                            |15       |40            |1           |
 |7        |2         |5      |2020-01-08 21:20:29.000    |2020-01-08 21:30:45.000  |10                            |25       |60            |1           |
 |8        |2         |4      |2020-01-09 23:54:33.000    |2020-01-10 00:15:02.000  |21                            |15       |93.6          |1           |
 |10       |1         |4      |2020-01-11 18:34:49.000    |2020-01-11 18:50:20.000  |16                            |10       |60            |2           |
 
### **Q5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled -  how much money does Pizza Runner have left over after these deliveries?**
```Query
with cte
as
(
select runner_id,
SUM(CASE WHEN pizza_id = 1 THEN 12 ELSE 10 END)-
SUM(CASE WHEN distance <> 0 THEN (distance * 0.30) ELSE 0 END)as Money_earned
from #updated_customer_orders c
join #updated_runner_orders r
on c.order_id=r.order_id
where distance <> 0
group by runner_id
)
select SUM(money_earned)as profit
from cte;
```
**Result**
|profit |
|-------|
|73.38  |

Pizza Runner's profit is 73.38$

### E. Bonus Questions
                
**If Danny wants to expand his range of pizzas - how would this impact the existing data design? Write an INSERT statement to 
demonstrate what would happen if a new Supreme pizza with all the toppings was added to the Pizza Runner menu?**
```Query
Insert into pizza_names values
(3,'Supreme_pizza');
insert pizza_recipes values
(3,'1,2,3,4,5,6,7,8,9,10,11,12');
select * from pizza_names
select * from pizza_recipes;
```
**Result**
|pizza_id |pizza_name   |
|---------|-------------|
|1        |Meatlovers   |
|2        |Vegetarian   |
|3        |Supreme_pizza|

|pizza_id    |toppings                 |
|------------|-------------------------|
|1           |1,2,3,4,5,6,8,10         |
|2           |4,6,7,9,11,12            |
|3           |1,2,3,4,5,6,,8,9,10,11,12|




  
  
  
  
  
  
  
  

