Cleanup of customer_orders:

Old table:
![image](https://github.com/user-attachments/assets/41885436-ab4d-4d7e-b366-5ca7740b3d05)


Comments:

```sql
CREATE TEMPORARY TABLE customer_orders_new AS
  SELECT 
    order_id, 
    customer_id, 
    pizza_id, 
    order_time,
    CASE
        WHEN exclusions IS null OR exclusions LIKE 'null' THEN ' '
        ELSE exclusions
        END AS exclusions,
    CASE
        WHEN extras IS NULL or extras LIKE 'null' THEN ' '
        ELSE extras
        END AS extras
  FROM pizza_runner.customer_orders;
```

Resulting table:
![image](https://github.com/user-attachments/assets/ed425b2c-2277-44bf-9102-b6d54253281c)

***
Cleanup of runners_orders

Old table:

![image](https://github.com/user-attachments/assets/1f88b882-dbaf-41a5-b242-ab47a62dd4fb)


Comments:

```sql
CREATE TEMPORARY TABLE runner_orders_new AS
SELECT 
  order_id, 
  runner_id,  
  CASE
	  WHEN pickup_time LIKE 'null' THEN ' '
	  ELSE pickup_time
	  END AS pickup_time,
  CASE
	  WHEN distance LIKE 'null' THEN ' '
	  WHEN distance LIKE '%km' THEN TRIM('km' from distance)
	  ELSE distance 
    END AS distance,
  CASE
	  WHEN duration LIKE 'null' THEN ' '
	  WHEN duration LIKE '%mins' THEN TRIM('mins' from duration)
	  WHEN duration LIKE '%minute' THEN TRIM('minute' from duration)
	  WHEN duration LIKE '%minutes' THEN TRIM('minutes' from duration)
	  ELSE duration
	  END AS duration,
  CASE
	  WHEN cancellation IS NULL or cancellation LIKE 'null' THEN ' '
	  ELSE cancellation
	  END AS cancellation
FROM pizza_runner.runner_orders;
```
Resulting table:
![image](https://github.com/user-attachments/assets/1873414e-6557-4034-b93e-f7901675d1ce)

***

Section A

Question 1: How many pizzas were ordered?

```sql
SELECT
COUNT(*) AS number_of_orders
FROM customer_orders_new;
```
***

Question 2: How many unique customer orders were made?

```sql
SELECT
COUNT(DISTINCT order_id) as unique_orders
FROM customer_orders_new;
```

***

Question 3: How many successful orders were delivered by each runner?

```sql
SELECT
	runner_id,
    COUNT(order_id) AS successful_deliveries
FROM runner_orders_new
WHERE cancellation LIKE ' ' OR cancellation LIKE ''
GROUP BY runner_id;
```

***

Question 4: How many of each type of pizza was delivered?

```sql
SELECT 
	p.pizza_name,
    COUNT(r.order_id) AS delivered_pizza
FROM runner_orders_new AS r
INNER JOIN customer_orders_new as c
	ON c.order_id = r.order_id
INNER JOIN pizza_runner.pizza_names as p
	ON p.pizza_id = c.pizza_id
WHERE cancellation LIKE ' ' OR cancellation LIKE '' -- filter for delivered orders
GROUP BY p.pizza_name
```

***

Question 5: How many Vegetarian and Meatlovers were ordered by each customer?

```sql
SELECT
	c.customer_id,
    COUNT(p.pizza_id) AS pizzas_ordered,
    p.pizza_name
FROM customer_orders_new AS C
INNER JOIN pizza_runner.pizza_names AS P
	ON c.pizza_id = p.pizza_id
GROUP BY c.customer_id, p.pizza_name
ORDER BY customer_id;
```

***

Question 6: What was the maximum number of pizzas delivered in a single order?

```sql
WITH maxcte AS(
  SELECT 
  	c.order_id,
  	COUNT(c.pizza_id) AS pizzas_ordered
  FROM customer_orders_new AS c
  INNER JOIN runner_orders_new AS r
  	ON c.order_id = r.order_id
  WHERE cancellation LIKE ' ' OR cancellation LIKE ''
  GROUP BY c.order_id
  
  )
  
 SELECT
 order_id,
 MAX(pizzas_ordered)
 FROM maxcte
 GROUP BY order_id
 ORDER BY MAX(pizzas_ordered) DESC
```

***

Question 7: For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

```sql
SELECT
	c.customer_id,
    SUM(CASE 
        	WHEN exclusions LIKE '%'  OR extras LIKE '%'  THEN 1
        	ELSE 0
        END) AS atleast_one_change,
    SUM(CASE
        	WHEN exclusions NOT LIKE '%'  AND extras NOT LIKE '%' THEN 1
        	ELSE 0
        END) AS no_changes
FROM customer_orders_new AS c
INNER JOIN runner_orders_new AS r
	ON c.order_id = r.order_id
WHERE cancellation LIKE ' ' OR cancellation LIKE ''
GROUP BY customer_id
ORDER BY customer_id;
```















