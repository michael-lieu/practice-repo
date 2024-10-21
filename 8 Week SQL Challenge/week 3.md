Question 1: How many customers has Foodie-Fi ever had?

```sql
SELECT 
COUNT(DISTINCT customer_id) AS number_of__unique_customers
FROM foodie_fi.subscriptions;
```

***

Question 2: What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value

```sql
SELECT 
DATE_TRUNC('month', s.start_date) AS month,
COUNT(s.customer_id)
FROM foodie_fi.subscriptions as s
INNER JOIN foodie_fi.plans as p ON s.plan_id = p.plan_id
WHERE p.plan_id = 0
GROUP BY month
ORDER BY month;
```

***

Question 3: What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name

```sql
SELECT 
p.plan_name AS plan_name,
COUNT(s.plan_id) AS plan_count
FROM foodie_fi.subscriptions as s
INNER JOIN foodie_fi.plans as p ON s.plan_id = p.plan_id
WHERE s.start_date >= '2021-01-01'
GROUP BY p.plan_name;
```

***

Question 4: What is the customer count and percentage of customers who have churned rounded to 1 decimal place?

```sql
SELECT 
COUNT(DISTINCT customer_id) AS churn_count,
ROUND(100.0*COUNT(customer_id)/
      (SELECT COUNT(DISTINCT customer_id)
       FROM foodie_fi.subscriptions),1)AS churn_percentage,
p.plan_name AS plan_name
FROM foodie_fi.subscriptions as s
INNER JOIN foodie_fi.plans as p ON s.plan_id = p.plan_id
WHERE plan_name = 'churn'
GROUP BY plan_name;
```

***

Question 5: How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?

```sql
WITH rankCTE AS(
SELECT
  p.plan_id,
  s.customer_id,
  ROW_NUMBER() OVER(PARTITION BY s.customer_id ORDER BY s.start_date) AS row_num
FROM foodie_fi.subscriptions as s
  INNER JOIN foodie_fi.plans as p ON s.plan_id = p.plan_id
  )
  
SELECT
  COUNT(CASE
    WHEN row_num = 2 AND plan_id = 4 THEN 1
    ELSE 0
    END) AS churn_count,
    
  ROUND(100*COUNT(CASE
    WHEN row_num = 2 AND plan_id = 4 THEN 1 ELSE 0 END)
        /(SELECT COUNT(DISTINCT customer_id) FROM foodie_fi.subscriptions)) AS churn_pct            
FROM rankCTE
WHERE plan_id = 4 AND row_num = 2
```

***

Question 6: What is the number and percentage of customer plans after their initial free trial?

```sql
WITH rankCTE AS(
SELECT
  p.plan_id,
  p.plan_name,
  s.customer_id,
  ROW_NUMBER() OVER(PARTITION BY s.customer_id ORDER BY s.start_date) AS row_num
FROM foodie_fi.subscriptions as s
  INNER JOIN foodie_fi.plans as p ON s.plan_id = p.plan_id
  )
  
SELECT
	plan_name,
    COUNT(customer_id),
    ROUND(100*COUNT(customer_id)/(SELECT COUNT(DISTINCT customer_id) FROM foodie_fi.subscriptions),1)
FROM rankCTE
WHERE row_num = 2
GROUP BY plan_name;
```

***

Question 7: What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?

```sql
WITH rankCTE AS(
  
SELECT 
  p.plan_id,
  s.customer_id,
  p.plan_name,
  ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY s.start_date DESC) AS row_num
  FROM foodie_fi.subscriptions as s
  INNER JOIN foodie_fi.plans as p ON p.plan_id=s.plan_id
  WHERE s.start_date <= '2020-12-31'
  )

SELECT
	plan_name,
    COUNT(customer_id) AS customer_count,
    ROUND(100.0*COUNT(customer_id)/(SELECT COUNT(DISTINCT customer_id) FROM rankCTE),1) AS customer_pct
FROM rankCTE
WHERE row_num = 1
GROUP BY plan_name;
```

***

Question 8: How many customers have upgraded to an annual plan in 2020?

```sql
SELECT 
COUNT(DISTINCT customer_id) AS customer_count
FROM foodie_fi.subscriptions
WHERE plan_id = 3
  AND start_date BETWEEN '2020-01-01' AND '2020-12-31';
```

***

Question 9: How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?

```sql
WITH annual_plan_date AS (
  SELECT
  customer_id,
  start_date AS annual_start
  FROM foodie_fi.subscriptions
  WHERE plan_id = 3 -- select plan ID for annual plan, here we are calculating start date of first annual plan
  ),
  
  trial_plan_date AS (
    SELECT
    customer_id,
    start_date AS trial_start
    FROM foodie_fi.subscriptions
    WHERE plan_id = 0 --select plan ID for trial plan
    )
    
    SELECT
    AVG ( a.annual_start - t.trial_start) AS avg_days
    FROM annual_plan_date AS a
    INNER JOIN trial_plan_date as t ON a.customer_id = t.customer_id;
```

***

Question 10: Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc) -- use of width_bucket to create bins

```sql
WITH annual_plan_date AS (
  SELECT
  customer_id,
  start_date AS annual_start
  FROM foodie_fi.subscriptions
  WHERE plan_id = 3 -- select plan ID for annual plan, here we are calculating start date of first annual plan
  ),
  
  trial_plan_date AS (
    SELECT
    customer_id,
    start_date AS trial_start
    FROM foodie_fi.subscriptions
    WHERE plan_id = 0 --select plan ID for trial plan
    ),
    
    bins AS (
      SELECT 
    WIDTH_BUCKET(a.annual_start - t.trial_start, 0, 365, 13) AS upgrade_days
  FROM trial_plan_date AS t
  JOIN annual_plan_date AS a
    ON t.customer_id = a.customer_id
)
    
    SELECT
     ((upgrade_days - 1) * 30 || ' - ' || upgrade_days * 30 || ' days') AS bin_width,
     COUNT(*) AS customer_count
    FROM bins
    GROUP BY upgrade_days
    ORDER BY upgrade_days;
```

***

Question 11: How many customers downgraded from a pro monthly to a basic monthly plan in 2020?

``` sql
WITH pro_month AS (
SELECT 
customer_id,
start_date as pro_monthly_start
FROM subscriptions
WHERE plan_id = 2
),

  basic_month AS (
  SELECT 
  customer_id,
  start_date as basic_monthly_start
  FROM subscriptions
  WHERE plan_id = 1
  )

SELECT 
p.customer_id,
pro_monthly_start,
basic_monthly_start
FROM PRO_MON AS p
INNER JOIN BASIC_MON as b ON p.customer_id = b.customer_id
WHERE pro_monthly_start < basic_monthly_start
AND DATE_PART('year',basic_monthly_start) = 2020;
```
























