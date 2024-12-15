# Pizza_-Sales_SQL
--Retrieve the total number of orders placed.-2135
select count(order_id) from orders;


--Calculate the total revenue generated from pizza  sales 
select sum (quantity*price) as total_revenue_generated 
from(
SELECT order_details.order_id,order_details.pizza_id,order_details.quantity,pizzas.price
FROM order_details left join pizzas 
on order_details.pizza_id=pizzas.pizza_id);


--Identify the highest-priced pizza.
select max (price) from pizzas;


--Identify the most common pizza size ordered.
/* select pizza_id, sum(quantity) as total_order_quantity from order_details group by pizza_id order by total_order_quantity desc */
select size, sum(quantity) as order_size_quantity from (Select pizzas.size, order_details.pizza_id, order_details.quantity
From order_details Left Join pizzas
On order_details.pizza_id=pizzas.pizza_id) group by size order by order_size_quantity desc;


--List the top 5 most ordered pizza types along with their quantities.
SELECT category , SUM(quantity) as quantity
FROM (
SELECT 
    order_details.quantity,
    order_details.pizza_id,
    order_details.order_id, 
    pizzas.pizza_type_id, 
    pizza_types.category
FROM 
    order_details
LEFT JOIN 
    pizzas 
    ON order_details.pizza_id = pizzas.pizza_id
LEFT JOIN 
    pizza_types 
    ON pizzas.pizza_type_id = pizza_types.pizza_type_id )
    GROUP BY category
    ORDER BY quantity DESC
    FETCH FIRST 5 ROWS ONLY;

--Determine the distribution of orders by hour of the day.
SELECT count(order_id) as orders, hour(time) as hour 
FROM orders
group by hour
order by hour asc;

--Join relevant tables to find the category-wise distribution of pizzas.
select category,  count (category) 
from pizza_types
group by category;

SELECT round(AVG(quantity),0) as Average_pizza_per_day
from (SELECT  orders.date, sum(order_details.quantity) as quantity
FROM order_details 
LEFT JOIN orders
ON order_details.order_id = orders.order_id
group by date )
as order_quantity ;

--Determine the top 3 most ordered pizza types based on revenue.
SELECT pizza_types.name, ROUND(SUM(order_details.quantity*pizzas.price),0) as Revenue
FROM order_details 
LEFT JOIN pizzas 
ON order_details.pizza_id=pizzas.pizza_id
LEFT JOIN pizza_types
ON pizzas.pizza_type_id= pizza_types.pizza_type_id
GROUP BY pizza_types.name
ORDER BY revenue DESC
FETCH FIRST 3 ROWS ONLY;

--Calculate the percentage contribution of each pizza type to total revenue.
SELECT pizza_types.category,
       ROUND(SUM(order_details.quantity*pizzas.price),0) as Revenue,
       ROUND(100* revenue/ (SELECT sum(pizzas.price*order_details.quantity) 
                            FROM order_details LEFT JOIN pizzas 
                            ON order_details.pizza_id=pizzas.pizza_id),2) 
                         as percent_contri
FROM order_details 
LEFT JOIN pizzas 
ON order_details.pizza_id=pizzas.pizza_id
LEFT JOIN pizza_types
ON pizzas.pizza_type_id= pizza_types.pizza_type_id
GROUP BY pizza_types.category
ORDER BY percent_contri DESC;

--Analyze the  revenue generated over time.
SELECT HOUR(time) as time_of_day,sum(price*quantity) as revenue
FROM order_details 
LEFT JOIN pizzas 
ON order_details.pizza_id=pizzas.pizza_id
LEFT JOIN orders
ON order_details.order_id=orders.order_id
group by time_of_day
order by time_of_day asc;
