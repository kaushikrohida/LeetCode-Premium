# Human Traffic of Stadium

## Problem Description

Table: Stadium

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| id            | int     |
| visit_date    | date    |
| people        | int     |
+---------------+---------+

- `visit_date` is the column with unique values for this table.
- Each row of this table contains the visit date and visit id to the stadium with the number of people during the visit.
- As the id increases, the date increases as well.

Write a solution to display the records with three or more rows with consecutive id's, and the number of people is greater than or equal to 100 for each.

Return the result table ordered by visit_date in ascending order.

## Example

### Input:
Stadium table:
+------+------------+-----------+
| id   | visit_date | people    |
+------+------------+-----------+
| 1    | 2017-01-01 | 10        |
| 2    | 2017-01-02 | 109       |
| 3    | 2017-01-03 | 150       |
| 4    | 2017-01-04 | 99        |
| 5    | 2017-01-05 | 145       |
| 6    | 2017-01-06 | 1455      |
| 7    | 2017-01-07 | 199       |
| 8    | 2017-01-09 | 188       |
+------+------------+-----------+

### Output:
+------+------------+-----------+
| id   | visit_date | people    |
+------+------------+-----------+
| 5    | 2017-01-05 | 145       |
| 6    | 2017-01-06 | 1455      |
| 7    | 2017-01-07 | 199       |
| 8    | 2017-01-09 | 188       |
+------+------------+-----------+

### Explanation:
The four rows with ids 5, 6, 7, and 8 have consecutive ids and each of them has >= 100 people attended. Note that row 8 was included even though the visit_date was not the next day after row 7.
The rows with ids 2 and 3 are not included because we need at least three consecutive ids.

## Solution

```sql
WITH cte as (
SELECT 
    s.id as id1,
    s1.id as id2,
    s2.id as id3 
FROM Stadium s 
JOIN Stadium s1 
    ON s.id = s1.id + 1 
JOIN Stadium s2 
    ON s.id = s2.id + 2  
WHERE 
    s.people >=100 
    and s1.people >= 100 
    and s2.people >= 100
),

cte2 as (
    select
        id1
    from cte
    union
    select
        id2
    from cte
    union 
    select
        id3
    from cte
)

select
    *
from Stadium
where
    id IN (select id1 from cte2)
order by visit_date
```

## Explanation

1. The first CTE (`cte`) finds all groups of three consecutive days where the number of people is >= 100 for each day.
2. The second CTE (`cte2`) combines all the ids from these groups, removing duplicates.
3. The final query selects all rows from the Stadium table where the id is in the list of ids from `cte2`, and orders the result by visit_date.

This solution efficiently handles the requirement of finding consecutive days with high attendance, including cases where there might be more than three consecutive days meeting the criteria.
