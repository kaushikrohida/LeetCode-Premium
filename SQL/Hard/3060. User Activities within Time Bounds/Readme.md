Table: Sessions

+---------------+----------+
| Column Name   | Type     |
+---------------+----------+
| user_id       | int      |
| session_start | datetime |
| session_end   | datetime |
| session_id    | int      |
| session_type  | enum     |
+---------------+----------+
session_id is column of unique values for this table.
session_type is an ENUM (category) type of (Viewer, Streamer).
This table contains user id, session start, session end, session id and session type.
Write a solution to find the the users who have had at least one consecutive session of the same type (either 'Viewer' or 'Streamer') with a maximum gap of 12 hours between sessions.

Return the result table ordered by user_id in ascending order.

The result format is in the following example.

 

Example:

Input: 
Sessions table:
+---------+---------------------+---------------------+------------+--------------+
| user_id | session_start       | session_end         | session_id | session_type | 
+---------+---------------------+---------------------+------------+--------------+
| 101     | 2023-11-01 08:00:00 | 2023-11-01 09:00:00 | 1          | Viewer       |  
| 101     | 2023-11-01 10:00:00 | 2023-11-01 11:00:00 | 2          | Streamer     |   
| 102     | 2023-11-01 13:00:00 | 2023-11-01 14:00:00 | 3          | Viewer       | 
| 102     | 2023-11-01 15:00:00 | 2023-11-01 16:00:00 | 4          | Viewer       | 
| 101     | 2023-11-02 09:00:00 | 2023-11-02 10:00:00 | 5          | Viewer       | 
| 102     | 2023-11-02 12:00:00 | 2023-11-02 13:00:00 | 6          | Streamer     | 
| 101     | 2023-11-02 13:00:00 | 2023-11-02 14:00:00 | 7          | Streamer     | 
| 102     | 2023-11-02 16:00:00 | 2023-11-02 17:00:00 | 8          | Viewer       | 
| 103     | 2023-11-01 08:00:00 | 2023-11-01 09:00:00 | 9          | Viewer       | 
| 103     | 2023-11-02 20:00:00 | 2023-11-02 23:00:00 | 10         | Viewer       | 
| 103     | 2023-11-03 09:00:00 | 2023-11-03 10:00:00 | 11         | Viewer       | 
+---------+---------------------+---------------------+------------+--------------+
Output: 
+---------+
| user_id |
+---------+
| 102     |
| 103     |
+---------+
Explanation:
- User ID 101 will not be included in the final output as they do not have any consecutive sessions of the same session type.
- User ID 102 will be included in the final output as they had two viewer sessions with session IDs 3 and 4, respectively, and the time gap between them was less than 12 hours.
- User ID 103 participated in two viewer sessions with a gap of less than 12 hours between them, identified by session IDs 10 and 11. Therefore, user 103 will be included in the final output.
Output table is ordered by user_id in increasing order.


```sql
with cte as (
select
    user_id,
    session_start,
    session_end,
    session_id,
    session_type,
    lead(session_start) over (partition by user_id, session_type order by session_start) as nxt_session_start
from Sessions
)

select distinct
    user_id
from cte
where
    timestampdiff(hour, session_end, nxt_session_start) <= 12
order by user_id
```