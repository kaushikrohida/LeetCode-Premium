Table: Friends

+-------------+------+
| Column Name | Type |
+-------------+------+
| user1       | int  |
| user2       | int  |
+-------------+------+
(user1, user2) is the primary key (combination of unique values) of this table.
Each row contains information about friendship where user1 and user2 are friends.
Write a solution to find the popularity percentage for each user on Meta/Facebook. The popularity percentage is defined as the total number of friends the user has divided by the total number of users on the platform, then converted into a percentage by multiplying by 100, rounded to 2 decimal places.

Return the result table ordered by user1 in ascending order.

The result format is in the following example.

 

Example 1:

Input: 
Friends table:
+-------+-------+
| user1 | user2 | 
+-------+-------+
| 2     | 1     | 
| 1     | 3     | 
| 4     | 1     | 
| 1     | 5     | 
| 1     | 6     |
| 2     | 6     | 
| 7     | 2     | 
| 8     | 3     | 
| 3     | 9     |  
+-------+-------+
Output: 
+-------+-----------------------+
| user1 | percentage_popularity |
+-------+-----------------------+
| 1     | 55.56                 |
| 2     | 33.33                 |
| 3     | 33.33                 |
| 4     | 11.11                 |
| 5     | 11.11                 |
| 6     | 22.22                 |
| 7     | 11.11                 |
| 8     | 11.11                 |
| 9     | 11.11                 |
+-------+-----------------------+
Explanation: 
There are total 9 users on the platform.
- User "1" has friendships with 2, 3, 4, 5 and 6. Therefore, the percentage popularity for user 1 would be calculated as (5/9) * 100 = 55.56.
- User "2" has friendships with 1, 6 and 7. Therefore, the percentage popularity for user 2 would be calculated as (3/9) * 100 = 33.33.
- User "3" has friendships with 1, 8 and 9. Therefore, the percentage popularity for user 3 would be calculated as (3/9) * 100 = 33.33.
- User "4" has friendships with 1. Therefore, the percentage popularity for user 4 would be calculated as (1/9) * 100 = 11.11.
- User "5" has friendships with 1. Therefore, the percentage popularity for user 5 would be calculated as (1/9) * 100 = 11.11.
- User "6" has friendships with 1 and 2. Therefore, the percentage popularity for user 6 would be calculated as (2/9) * 100 = 22.22.
- User "7" has friendships with 2. Therefore, the percentage popularity for user 7 would be calculated as (1/9) * 100 = 11.11.
- User "8" has friendships with 3. Therefore, the percentage popularity for user 8 would be calculated as (1/9) * 100 = 11.11.
- User "9" has friendships with 3. Therefore, the percentage popularity for user 9 would be calculated as (1/9) * 100 = 11.11.
user1 is sorted in ascending order.



```sql
with cte as (
select
    user1,
    user2
from Friends
union all
select
    user2 as user1,
    user1 as user2
from Friends
order by user1, user2
)

select
    c.user1,
    round(100 * (count(distinct c.user2) / (count(c.user1) over ())), 2) as percentage_popularity
from cte c
group by 1
order by 1
```