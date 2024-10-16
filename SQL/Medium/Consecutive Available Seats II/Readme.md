# Consecutive Available Seats II

## Problem Description

This is a common pattern in SQL called the "gaps and islands" problem. The gaps and islands problem is about spotting continuous data groups ("islands") and places where data is missing ("gaps") in a sequence.

Write a solution to find the length of the longest consecutive sequence of available seats in the cinema.

## Table: Cinema

| Column Name | Type |
|-------------|------|
| seat_id     | int  |
| free        | bool |

- seat_id is an auto-increment column for this table.
- Each row of this table indicates whether the ith seat is free or not. 1 means free while 0 means occupied.

## Notes

- There will always be at most one longest consecutive sequence.
- If there are multiple consecutive sequences with the same length, include all of them in the output.
- Return the result table ordered by first_seat_id in ascending order.

## Example

### Input:

Cinema table:

| seat_id | free |
|---------|------|
| 1       | 1    |
| 2       | 0    |
| 3       | 1    |
| 4       | 1    |
| 5       | 1    |

### Output:

| first_seat_id | last_seat_id | consecutive_seats_len |
|---------------|--------------|------------------------|
| 3             | 5            | 3                      |

### Explanation:

The longest consecutive sequence of available seats starts from seat 3 and ends at seat 5 with a length of 3.

## Solution

```sql
WITH consecutive AS (
    SELECT
        seat_id,
        free,
        SUM(free) OVER (ORDER BY seat_id) AS cumulative_sum,
        ROW_NUMBER() OVER (ORDER BY seat_id) AS row_num
    FROM Cinema
),
groupings AS (
    SELECT
        MIN(seat_id) AS first_seat_id,
        MAX(seat_id) AS last_seat_id,
        MAX(seat_id) - MIN(seat_id) + 1 AS consecutive_seats_len
    FROM consecutive
    WHERE free = 1
    GROUP BY row_num - cumulative_sum
)

SELECT *
FROM groupings
WHERE 
    consecutive_seats_len IN (
        SELECT
            MAX(consecutive_seats_len)
        FROM groupings
    )
ORDER BY first_seat_id
