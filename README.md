test


```sql

select
customer_id,
sum(price)
    
from dannys_diner.sales as s
inner join dannys_diner.menu as m 
    on s.product_id = m.product_id
group by customer_id
```
