Table: Tasks

+---------------+----------+
| Column Name   | Type     |
+---------------+----------+
| task_id       | int      |
| employee_id   | int      |
| start_time    | datetime |
| end_time      | datetime |
+---------------+----------+
(task_id, employee_id) is the primary key for this table.
Each row in this table contains the task identifier, the employee identifier, and the start and end times of each task.
Write a solution to find the total duration of tasks for each employee and the maximum number of concurrent tasks an employee handled at any point in time. The total duration should be rounded down to the nearest number of full hours.

Return the result table ordered by employee_id ascending order.

The result format is in the following example.

 

Example:

Input:

Tasks table:

+---------+-------------+---------------------+---------------------+
| task_id | employee_id | start_time          | end_time            |
+---------+-------------+---------------------+---------------------+
| 1       | 1001        | 2023-05-01 08:00:00 | 2023-05-01 09:00:00 |
| 2       | 1001        | 2023-05-01 08:30:00 | 2023-05-01 10:30:00 |
| 3       | 1001        | 2023-05-01 11:00:00 | 2023-05-01 12:00:00 |
| 7       | 1001        | 2023-05-01 13:00:00 | 2023-05-01 15:30:00 |
| 4       | 1002        | 2023-05-01 09:00:00 | 2023-05-01 10:00:00 |
| 5       | 1002        | 2023-05-01 09:30:00 | 2023-05-01 11:30:00 |
| 6       | 1003        | 2023-05-01 14:00:00 | 2023-05-01 16:00:00 |
+---------+-------------+---------------------+---------------------+
Output:

+-------------+------------------+----------------------+
| employee_id | total_task_hours | max_concurrent_tasks |
+-------------+------------------+----------------------+
| 1001        | 6                | 2                    |
| 1002        | 2                | 2                    |
| 1003        | 2                | 1                    |
+-------------+------------------+----------------------+
Explanation:

For employee ID 1001:
Task 1 and Task 2 overlap from 08:30 to 09:00 (30 minutes).
Task 7 has a duration of 150 minutes (2 hours and 30 minutes).
Total task time: 60 (Task 1) + 120 (Task 2) + 60 (Task 3) + 150 (Task 7) - 30 (overlap) = 360 minutes = 6 hours.
Maximum concurrent tasks: 2 (during the overlap period).
For employee ID 1002:
Task 4 and Task 5 overlap from 09:30 to 10:00 (30 minutes).
Total task time: 60 (Task 4) + 120 (Task 5) - 30 (overlap) = 150 minutes = 2 hours and 30 minutes.
Total task hours (rounded down): 2 hours.
Maximum concurrent tasks: 2 (during the overlap period).
For employee ID 1003:
No overlapping tasks.
Total task time: 120 minutes = 2 hours.
Maximum concurrent tasks: 1.
Note: Output table is ordered by employee_id in ascending order.


```sql
with cte as(
select
    t1.employee_id,
    sum(if(t1.task_id = t2.task_id, timestampdiff(minute, t1.start_time, t1.end_time), - timestampdiff(minute, t2.start_time, t1.end_time))) as tasks_hours,
    count(t2.task_id) as concurrent_tasks
from Tasks t1
Join Tasks t2
    on t1.employee_id = t2.employee_id
    and t1.start_time <= t2.start_time
    and t2.start_time < t1.end_time
group by 1, t1.task_id
)

select
    employee_id,
    floor(sum(tasks_hours)/60) as total_task_hours,
    max(concurrent_tasks) as max_concurrent_tasks
from cte
group by 1
order by 1
```