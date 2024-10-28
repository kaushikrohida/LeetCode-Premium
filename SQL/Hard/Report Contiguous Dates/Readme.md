# Report Contiguous Dates

## Tables

### Table: Failed

+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| fail_date    | date    |
+--------------+---------+
fail_date is the primary key (column with unique values) for this table.  
This table contains the days of failed tasks.

### Table: Succeeded

+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| success_date | date    |
+--------------+---------+
success_date is the primary key (column with unique values) for this table.  
This table contains the days of succeeded tasks.

## Problem Statement

A system is running one task every day. Every task is independent of the previous tasks. The tasks can fail or succeed.

Write a solution to report the `period_state` for each continuous interval of days in the period from `2019-01-01` to `2019-12-31`.

`period_state` is 'failed' if tasks in this interval failed or 'succeeded' if tasks in this interval succeeded. Interval of days are retrieved as `start_date` and `end_date`.

Return the result table ordered by `start_date`.

## Example

### Input: 

**Failed table:**
+-------------------+
| fail_date         |
+-------------------+
| 2018-12-28        |
| 2018-12-29        |
| 2019-01-04        |
| 2019-01-05        |
+-------------------+

**Succeeded table:**
+-------------------+
| success_date      |
+-------------------+
| 2018-12-30        |
| 2018-12-31        |
| 2019-01-01        |
| 2019-01-02        |
| 2019-01-03        |
| 2019-01-06        |
+-------------------+

### Output: 
+--------------+--------------+--------------+
| period_state | start_date   | end_date     |
+--------------+--------------+--------------+
| succeeded    | 2019-01-01   | 2019-01-03   |
| failed       | 2019-01-04   | 2019-01-05   |
| succeeded    | 2019-01-06   | 2019-01-06   |
+--------------+--------------+--------------+

### Explanation: 
The report ignored the system state in 2018 as we care about the system in the period 2019-01-01 to 2019-12-31.  
From 2019-01-01 to 2019-01-03 all tasks succeeded and the system state was "succeeded".  
From 2019-01-04 to 2019-01-05 all tasks failed and the system state was "failed".  
From 2019-01-06 to 2019-01-06 all tasks succeeded and the system state was "succeeded".

## SQL Solution

```sql
with cte as (
        select
            fail_date as dates,
            'failed' as period_state,
            rank() over (order by fail_date) as rk
        from Failed
        where fail_date between '2019-01-01' and '2019-12-31'
        union all 
        select
            success_date as dates,
            'succeeded' as period_state,
            rank() over (order by success_date) as rk
        from Succeeded
        where success_date between '2019-01-01' and '2019-12-31'
)

select
    period_state,
    min(dates) as start_date,
    max(dates) as end_date
from (
    select
        dates,
        rank() over (order by dates) as orank,
        period_state,
        rk,
        (rank() over (order by dates) - rk) as inv
    from cte
) a
group by inv, period_state
order by 2
```