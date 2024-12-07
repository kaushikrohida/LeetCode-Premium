Your task is to analyze the effectiveness of Amazon's notifications in driving user engagement and conversions, considering the user purchase data. A purchase is considered to be associated with a notification if the purchase happens within the timeframe of earliest of below 2 events:
1-  2 hours from notification delivered time
2-  Next notification delivered time.


Each notification is sent for a particular product id but a customer may purchase same or another product. Considering these rules write an SQL to find number of purchases associated with each notification for same product or a different product in 2 separate columns, display the output in ascending order of notification id.

 


Table:notifications
+-----------------+------------+
| COLUMN_NAME     | DATA_TYPE  |
+-----------------+------------+
| notification_id | int        |
| delivered_at    | datetime   |
| product_id      | varchar(2) |
+-----------------+------------+

Table:purchases 
+--------------------+------------+
| COLUMN_NAME        | DATA_TYPE  |
+--------------------+------------+
| product_id         | varchar(2) |
| purchase_timestamp | datetime   |
| user_id            | int        |
+--------------------+------------+


```sql
with cte as (
select
	notification_id,
  	product_id,
    delivered_at,
    coalesce(least(addtime(delivered_at, '02:00:00'), lead(delivered_at, 1) over (order by notification_id)), addtime(delivered_at, '02:00:00')) valid_time
from notifications
)

select
	c.notification_id,
	count(case when c.product_id = p.product_id then 1 else null end) as same_product_purchases,
    count(case when c.product_id <> p.product_id then 1 else null end) as different_product_purchases
from cte c
left join purchases p
	on p.purchase_timestamp between c.delivered_at and c.valid_time
group by 1
order by 1
```