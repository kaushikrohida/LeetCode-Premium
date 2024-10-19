# Server Utilization Time

## Problem Description

Given a table `Servers` with information about server status changes, write a SQL solution to find the total time when servers were running. The output should be rounded down to the nearest number of full days.

## Table Structure

Table: Servers

| Column Name    | Type     |
|----------------|----------|
| server_id      | int      |
| status_time    | datetime |
| session_status | enum     |

- (server_id, status_time, session_status) is the primary key for this table.
- session_status is an ENUM type of ('start', 'stop').
- Each row contains server_id, status_time, and session_status.

## Example

### Input:

Servers table:

| server_id | status_time         | session_status |
|-----------|---------------------|----------------|
| 3         | 2023-11-04 16:29:47 | start          |
| 3         | 2023-11-05 01:49:47 | stop           |
| 3         | 2023-11-25 01:37:08 | start          |
| 3         | 2023-11-25 03:50:08 | stop           |
| 1         | 2023-11-13 03:05:31 | start          |
| 1         | 2023-11-13 11:10:31 | stop           |
| 4         | 2023-11-29 15:11:17 | start          |
| 4         | 2023-11-29 15:42:17 | stop           |
| 4         | 2023-11-20 00:31:44 | start          |
| 4         | 2023-11-20 07:03:44 | stop           |
| 1         | 2023-11-20 00:27:11 | start          |
| 1         | 2023-11-20 01:41:11 | stop           |
| 3         | 2023-11-04 23:16:48 | start          |
| 3         | 2023-11-05 01:15:48 | stop           |
| 4         | 2023-11-30 15:09:18 | start          |
| 4         | 2023-11-30 20:48:18 | stop           |
| 4         | 2023-11-25 21:09:06 | start          |
| 4         | 2023-11-26 04:58:06 | stop           |
| 5         | 2023-11-16 19:42:22 | start          |
| 5         | 2023-11-16 21:08:22 | stop           |

### Output:

| total_uptime_days |
|-------------------|
| 1                 |

## Explanation

- Server ID 3: ~13.48 hours
- Server ID 1: ~9.23 hours
- Server ID 4: ~20.52 hours
- Server ID 5: ~1.43 hours

The accumulated runtime for all servers totals approximately 44.46 hours, equivalent to one full day plus some additional hours. However, since we consider only full days, the final output is rounded to 1 full day.

## Solution

```sql
WITH cte AS (
    SELECT
        server_id,
        status_time,
        session_status,
        LEAD(status_time, 1) OVER (PARTITION BY server_id ORDER BY status_time ASC, session_status ASC) AS status_time2
    FROM Servers
)

SELECT
    FLOOR(SUM(TIMESTAMPDIFF(SECOND, status_time, status_time2))/(24*60*60)) AS total_uptime_days
FROM cte
WHERE session_status = 'start'
```

### Explanation:

1. We use a Common Table Expression (CTE) to create a derived table that includes the next status time for each row.
2. The LEAD function is used to get the next status time for each server.
3. We then calculate the time difference between each 'start' and its corresponding 'stop' time in seconds.
4. The total seconds are summed up and divided by the number of seconds in a day (24*60*60) to get the total days.
5. FLOOR is used to round down to the nearest whole day.

This solution efficiently calculates the total uptime across all servers and returns the result in full days.
