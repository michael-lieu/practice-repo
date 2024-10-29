PART A


Question 1: How many unique nodes are there on the Data Bank system?

``` sql
SELECT 
COUNT(DISTINCT node_id) 
FROM data_bank.customer_nodes;
```

***

Question 2: What is the number of nodes per region?

```sql
SELECT 
r.region_name,
COUNT(DISTINCT node_id) as node_count
FROM data_bank.customer_nodes as cn
INNER JOIN data_bank.regions as r ON cn.region_id = r.region_id
GROUP BY r.region_name;
```

***

Question 3: How many customers are allocated to each region?

```sql
SELECT 
r.region_name,
COUNT(DISTINCT customer_id) as customer_count
FROM data_bank.customer_nodes as cn
INNER JOIN data_bank.regions as r ON cn.region_id = r.region_id
GROUP BY r.region_name;
```

Question 4: How many days on average are customers reallocated to a different node?

```sql
WITH days_spent_in_node AS(
  
  SELECT
  customer_id,
  node_id,
  SUM(end_date - start_date) AS days_in_node
  FROM data_bank.customer_nodes
  WHERE end_date != '9999-12-31'
  GROUP BY customer_id, node_id
  )
  
SELECT 
ROUND(AVG(days_in_node),0) AS avg_days_reallocation
FROM days_spent_in_node;
```

***

PART B

Question 1


















