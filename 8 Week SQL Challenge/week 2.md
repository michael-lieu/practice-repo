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
CREATE TEMP TABLE runner_orders_new AS
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






