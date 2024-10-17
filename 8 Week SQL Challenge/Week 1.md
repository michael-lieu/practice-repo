**1. What is the total amount each customer spent at the restaurant?**

**Notes: Inner join used to merge the sales and menu tables, as we are retrieving the customer_id from the sales table and the price from the menu table. SUM aggregation used to determine the total amount spent by each customer and then the results are grouped by the customer_id**

```sql
SELECT
    customer_id,
    SUM(price) AS total_amount
FROM dannys_diner.sales AS s
INNER JOIN dannys_diner.menu AS m 
    ON s.product_id = m.product_id
GROUP BY customer_id
ORDER BY customer_id;
```
**Result:**
| customer_id | total_amount |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

***

**2. How many days has each customer visited the restaurant?**\

**Notes: Use a distinct count to determine the number of visits to the restaurant as there are duplicate order dates of the same date if the customer bought more than one item on that particular day.**
   
```sql
SELECT
    customer_id,
    COUNT(DISTINCT order_date) AS days_visited
FROM dannys_diner.sales
GROUP BY customer_id
ORDER BY customer_id;
```
**Result:**
| customer_id | days_visited |
| ----------- | ----------- |
| A           | 4          |
| B           | 6          |
| C           | 2          |

***

**3. What was the first item from the menu purchased by each customer?**\

**Notes: create CTE to rank the purchases of each customer, ordered by the order date in ascending order and then selecting the appropriate columns with rank equal to one to find the first item purchased by each customer**

```sql
WITH rankCTE AS (
  SELECT 
  s.customer_id,
  s.order_date,
  m.product_name,
  DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date ASC) AS product_rank
  FROM dannys_diner.sales AS s
  INNER JOIN dannys_diner.menu AS m
  ON s.product_id = m.product_id
 )
  
SELECT
    customer_id,
    product_name
FROM rankCTE
WHERE product_rank = 1
GROUP BY customer_id, product_name;
```
**Result:**
| customer_id | product_name | 
| ----------- | ----------- |
| A           | curry        | 
| A           | sushi        | 
| B           | curry        | 
| C           | ramen        |

***

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**\

**Notes: Inner join used to connect the sales with the menu to obtain the product names, count aggregation used to count the amount of times each product was purchased and then group by the product name and limiting to one to find out the most purchased item.**

```sql
SELECT
    m.product_name,
    COUNT(s.product_id) AS times_purchased
FROM dannys_diner.sales as s
INNER JOIN dannys_diner.menu as m
    ON s.product_id = m.product_id
GROUP BY m.product_name
ORDER BY times_purchased desc
LIMIT 1;
```
**Results:**
| product_name | times_purchased | 
| ----------- | ----------- |
| ramen       | 8 |

***

**5. Which item was the most popular for each customer?**\

**Notes: create a CTE to rank the customers count of orders to find their most ordered products and then select the appropriate columns to answer the question.**

```sql
WITH rankCTE AS (
  SELECT 
  s.customer_id,
  m.product_name,
  COUNT(s.order_date) AS orders,
  DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY COUNT(s.order_date) DESC) AS favourite_rank
  FROM dannys_diner.sales AS s
  INNER JOIN dannys_diner.menu AS m
  ON s.product_id = m.product_id
  GROUP BY s.customer_id, m.product_name
 )
 
 SELECT
 	customer_id,
 	product_name,
 	orders
 FROM rankCTE
 WHERE favourite_rank=1;
```
**Result:**
| customer_id | product_name | orders |
| ----------- | ---------- |------------  |
| A           | ramen        |  3   |
| B           | ramen        |  2   |
| B           | curry        |  2   |
| B           | sushi        |  2   |
| C           | ramen        |  3   |

***

**6. Which item was purchased first by the customer after they became a member?**\
**notes: create a CTE to rank the customer's orders after becoming a member ordered by the order date, then select the appropriate columns with rank equal to one for the desired result**

```sql
WITH firstCTE AS(
  SELECT
      s.customer_id,
      m.product_name,
      s.order_date,
      mem.join_date,
  	  DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY order_date) AS firstrank
  FROM dannys_diner.sales AS s
  INNER JOIN dannys_diner.menu AS m
      ON s.product_id = m.product_id
  INNER JOIN dannys_diner.members AS mem
      ON s.customer_id = mem.customer_id
  WHERE order_date >= join_date
  )
  
SELECT
	customer_id,
    product_name
FROM firstCTE
WHERE firstrank=1;
```
**Results:**
| customer_id | product_name |
| ----------- | ---------- |
| A           | ramen        |
| B           | sushi        |

***

**7. Which item was purchased just before the customer became a member?**\
**notes:similar to question 6 but just needed to make the order by date descending and change the where clause to make it filter for before the member join date**
   
```sql
WITH firstCTE AS(
  SELECT
      s.customer_id,
      m.product_name,
      s.order_date,
      mem.join_date,
  	  DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY order_date DESC) AS firstrank
  FROM dannys_diner.sales AS s
  INNER JOIN dannys_diner.menu AS m
      ON s.product_id = m.product_id
  INNER JOIN dannys_diner.members AS mem
      ON s.customer_id = mem.customer_id
  WHERE order_date < join_date
  )
  
SELECT
	customer_id,
    product_name,
    order_date
FROM firstCTE
WHERE firstrank=1;
```
**Results:**
| customer_id | product_name | order_date |
| ----------- | ---------- |------------  |
| A           | sushi        |  2021-01-01   |
| A          | curry       |  2021-01-01   |
| B           | sushi        |  2021-01-04   |

***


**8. What is the total items and amount spent for each member before they became a member?**\
**notes:perform inner join on all three tables do that we can use aggregations of count on the product_id to determine the total items and sum of price so we can calculate the total amount spent, filter for order dates before join dates and group by customer_id to obtain the desired results.**
   
```sql
SELECT
	s.customer_id,
    COUNT(s.product_id) AS total_products,
    SUM(m.price) AS total_spent
FROM dannys_diner.sales AS s
INNER JOIN dannys_diner.menu AS m
	ON s.product_id = m.product_id
INNER JOIN dannys_diner.members AS mem
	ON s.customer_id = mem.customer_id
WHERE order_date < join_date
GROUP BY s.customer_id 
ORDER BY s.customer_id ASC;
```

**Results:**
| customer_id | total_products | total_spent |
| ----------- | ---------- |----------  |
| A           | 2 |  25       |
| B           | 3 |  40       |

***

**9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**\
    
**notes: Use a CASE WHEN argument to multiply the price by 2*10 when the product_name is sushi, else multiply price by 10 and use a sum aggregation to calculate the total points.**

```sql
SELECT
	s.customer_id,
    SUM(CASE
        WHEN m.product_name = 'sushi' THEN price * 2 * 10
        ELSE price * 10
        END) AS total_points
FROM dannys_diner.sales AS s
INNER JOIN dannys_diner.menu AS m
	ON s.product_id = m.product_id
GROUP BY customer_id
ORDER BY customer_id;
```
**Results:**
| customer_id | total_points | 
| ----------- | ---------- |
| A           | 860 |
| B           | 940 |
| C           | 360 |

***

**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**\
    
**notes:Building upon the previous question, we add another WHEN clause into the SUM aggregation so that the points are also doubled when the order date is in the first week of becoming a member. We additionally use the DATE_TRUNC function in order to calculate the amount of points accumulated up until the end of January in 2021.**

```sql
SELECT
	s.customer_id,
    SUM(CASE
        WHEN m.product_name = 'sushi' THEN price * 2 * 10
        WHEN order_date BETWEEN mem.join_date AND (join_date + interval '6 days') THEN price * 2 * 10
        ELSE price * 10
        END) AS total_points
FROM dannys_diner.sales AS s
INNER JOIN dannys_diner.menu AS m
	ON s.product_id = m.product_id
INNER JOIN dannys_diner.members as mem
	ON s.customer_id = mem.customer_id
WHERE DATE_TRUNC('month', order_date) = '2021-01-01'
GROUP BY s.customer_id
ORDER BY s.customer_id;
```

**Results:**
| customer_id | total_points | 
| ----------- | ---------- |
| A           | 1370 |
| B           | 320 |









