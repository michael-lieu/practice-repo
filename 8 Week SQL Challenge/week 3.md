Question 1: How many customers has Foodie-Fi ever had?

SELECT 
COUNT(DISTINCT customer_id) AS number_of__unique_customers
FROM foodie_fi.subscriptions;
