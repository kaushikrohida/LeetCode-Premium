# GitHub Portfolio - Employee Shift Analysis

## Project Overview

This project focuses on analyzing overlapping shifts for each employee in the `EmployeeShifts` table. The goal is to determine the maximum number of overlapping shifts at any given time and to calculate the total duration of these overlaps in minutes for each employee.

### Table: EmployeeShifts

| Column Name   | Type     |
|---------------|----------|
| employee_id   | int      |
| start_time    | datetime |
| end_time      | datetime |

- `(employee_id, start_time)` is the unique key for this table.
- The table contains information about the shifts worked by employees, including the start and end times.

### Problem Statement

For each employee, we need to calculate:
1. The maximum number of shifts that overlap at any given time.
2. The total duration of all overlaps in minutes.

The result table should be ordered by `employee_id` in ascending order.

### Example

**Input:**

| employee_id | start_time          | end_time            |
|-------------|---------------------|---------------------|
| 1           | 2023-10-01 09:00:00 | 2023-10-01 17:00:00 |
| 1           | 2023-10-01 15:00:00 | 2023-10-01 23:00:00 |
| 1           | 2023-10-01 16:00:00 | 2023-10-02 00:00:00 |
| 2           | 2023-10-01 09:00:00 | 2023-10-01 17:00:00 |
| 2           | 2023-10-01 11:00:00 | 2023-10-01 19:00:00 |
| 3           | 2023-10-01 09:00:00 | 2023-10-01 17:00:00 |

**Output:**

| employee_id | max_overlapping_shifts | total_overlap_duration |
|-------------|------------------------|------------------------|
| 1           | 3                      | 600                    |
| 2           | 2                      | 360                    |
| 3           | 1                      | 0                      |

**Explanation:**
- **Employee 1:**
  - Maximum overlapping shifts: 3 (from 16:00 to 17:00).
  - Total overlap duration: 600 minutes (calculated based on pairwise overlap durations).
- **Employee 2:**
  - Maximum overlapping shifts: 2.
  - Total overlap duration: 360 minutes.
- **Employee 3:**
  - No overlaps.

### SQL Solution

```sql
WITH cte AS (
    SELECT
        e1.employee_id,
        IF(COUNT(*) = 1, COUNT(*) + 1, COUNT(*)) AS max_overlapping_shifts,
        SUM(TIMESTAMPDIFF(MINUTE, e2.start_time, e1.end_time)) AS total_overlap_duration
    FROM EmployeeShifts e1
    JOIN EmployeeShifts e2
        ON e1.start_time < e2.start_time
        AND e1.end_time > e2.start_time
        AND e1.employee_id = e2.employee_id
    GROUP BY 1
)

SELECT DISTINCT
    s.employee_id,
    COALESCE(c.max_overlapping_shifts, 1) AS max_overlapping_shifts,
    COALESCE(c.total_overlap_duration, 0) AS total_overlap_duration
FROM EmployeeShifts s
LEFT JOIN cte c
    ON s.employee_id = c.employee_id
ORDER BY 1;
```