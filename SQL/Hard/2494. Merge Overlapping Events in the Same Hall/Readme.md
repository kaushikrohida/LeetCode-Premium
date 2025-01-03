Table: HallEvents

+-------------+------+
| Column Name | Type |
+-------------+------+
| hall_id     | int  |
| start_day   | date |
| end_day     | date |
+-------------+------+
This table may contain duplicates rows.
Each row of this table indicates the start day and end day of an event and the hall in which the event is held.
 

Write a solution to merge all the overlapping events that are held in the same hall. Two events overlap if they have at least one day in common.

Return the result table in any order.

The result format is in the following example.

 

Example 1:

Input: 
HallEvents table:
+---------+------------+------------+
| hall_id | start_day  | end_day    |
+---------+------------+------------+
| 1       | 2023-01-13 | 2023-01-14 |
| 1       | 2023-01-14 | 2023-01-17 |
| 1       | 2023-01-18 | 2023-01-25 |
| 2       | 2022-12-09 | 2022-12-23 |
| 2       | 2022-12-13 | 2022-12-17 |
| 3       | 2022-12-01 | 2023-01-30 |
+---------+------------+------------+
Output: 
+---------+------------+------------+
| hall_id | start_day  | end_day    |
+---------+------------+------------+
| 1       | 2023-01-13 | 2023-01-17 |
| 1       | 2023-01-18 | 2023-01-25 |
| 2       | 2022-12-09 | 2022-12-23 |
| 3       | 2022-12-01 | 2023-01-30 |
+---------+------------+------------+
Explanation: There are three halls.
Hall 1:
- The two events ["2023-01-13", "2023-01-14"] and ["2023-01-14", "2023-01-17"] overlap. We merge them in one event ["2023-01-13", "2023-01-17"].
- The event ["2023-01-18", "2023-01-25"] does not overlap with any other event, so we leave it as it is.
Hall 2:
- The two events ["2022-12-09", "2022-12-23"] and ["2022-12-13", "2022-12-17"] overlap. We merge them in one event ["2022-12-09", "2022-12-23"].
Hall 3:
- The hall has only one event, so we return it. Note that we only consider the events of each hall separately.



```sql
with cte1 as (
    select
        hall_id,
        start_day,
        end_day,
        max(end_day) over (partition by hall_id order by start_day) as max_end
    from HallEvents
),
cte2 as (
    select
        hall_id,
        start_day,
        end_day,
        case
            when start_day <= lag(max_end, 1, 0) over (partition by hall_id order by start_day) then 0
            else 1
        end as range_start
    from cte1
),
cte3 as (
    select
        hall_id,
        start_day,
        end_day,
        range_start,
        sum(range_start) over (partition by hall_id order by start_day) as range_flag
    from cte2
)

select
    hall_id,
    min(start_day) as start_day,
    max(end_day) as end_day
from cte3
group by hall_id, range_flag
```