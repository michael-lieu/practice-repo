Cleanup of customer_orders:

'''sql
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
'''

Resulting table:
![image](https://github.com/user-attachments/assets/ed425b2c-2277-44bf-9102-b6d54253281c)
