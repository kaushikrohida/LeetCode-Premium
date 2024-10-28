# UserActivity Table

The `UserActivity` table contains information about the activities performed by users over a period of time. Each record includes the username, the activity performed, and the start and end dates of that activity.

## Table Structure

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| username      | varchar |
| activity      | varchar |
| startDate     | Date    |
| endDate       | Date    |
+---------------+---------+

### Notes
- This table may contain duplicate rows.
- A user cannot perform more than one activity at the same time.

## Problem Statement

Write a solution to show the second most recent activity of each user. If the user only has one activity, return that one.

### Example Input

UserActivity table:
+------------+--------------+-------------+-------------+
| username   | activity     | startDate   | endDate     |
+------------+--------------+-------------+-------------+
| Alice      | Travel       | 2020-02-12  | 2020-02-20  |
| Alice      | Dancing      | 2020-02-21  | 2020-02-23  |
| Alice      | Travel       | 2020-02-24  | 2020-02-28  |
| Bob        | Travel       | 2020-02-11  | 2020-02-18  |
+------------+--------------+-------------+-------------+

### Example Output

+------------+--------------+-------------+-------------+
| username   | activity     | startDate   | endDate     |
+------------+--------------+-------------+-------------+
| Alice      | Dancing      | 2020-02-21  | 2020-02-23  |
| Bob        | Travel       | 2020-02-11  | 2020-02-18  |
+------------+--------------+-------------+-------------+

### Explanation
- The most recent activity of Alice is Travel from 2020-02-24 to 2020-02-28, before that she was dancing from 2020-02-21 to 2020-02-23.
- Bob only has one record, so we just take that one.

## SQL Solution

```sql
with cte as (
    select
        *,
        row_number() over (partition by username order by startDate desc) rn,
        count(activity) over (partition by username) count_n
    from UserActivity
)

select
    username,
    activity,
    startDate,
    endDate
from cte
where rn = 2 or count_n < 2
```

