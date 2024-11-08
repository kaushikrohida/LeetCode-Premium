# Friday Spending Analysis in November 2023

## Table: Purchases

This project involves analyzing user spending data for Fridays in the month of November 2023. Below is the table schema used:

| Column Name   | Type |
|---------------|------|
| user_id       | int  |
| purchase_date | date |
| amount_spend  | int  |

**Primary Key**: (user_id, purchase_date, amount_spend)

- `purchase_date` will range from November 1, 2023, to November 30, 2023, inclusive of both dates.
- Each row contains `user_id`, `purchase_date`, and `amount_spend`.

## Problem Statement

We need to calculate the total spending by users on each Friday of every week in November 2023. If there are no purchases on a particular Friday, the value should be considered as 0.

The result table should be ordered by `week_of_month` in ascending order.

### Example

**Input**:

| user_id | purchase_date | amount_spend |
|---------|---------------|--------------|
| 11      | 2023-11-07    | 1126         |
| 15      | 2023-11-30    | 7473         |
| 17      | 2023-11-14    | 2414         |
| 12      | 2023-11-24    | 9692         |
| 8       | 2023-11-03    | 5117         |
| 1       | 2023-11-16    | 5241         |
| 10      | 2023-11-12    | 8266         |
| 13      | 2023-11-24    | 12000        |

**Output**:

| week_of_month | purchase_date | total_amount |
|---------------|---------------|--------------|
| 1             | 2023-11-03    | 5117         |
| 2             | 2023-11-10    | 0            |
| 3             | 2023-11-17    | 0            |
| 4             | 2023-11-24    | 21692        |

### Explanation

- **Week 1**: Total transactions on `2023-11-03` were $5,117.
- **Week 2**: No transactions on `2023-11-10`, resulting in a value of `0`.
- **Week 3**: No transactions on `2023-11-17`, resulting in a value of `0`.
- **Week 4**: Total transactions on `2023-11-24` were $21,692.

## Solution

The solution below uses a Common Table Expression (CTE) to generate all the dates in November 2023, then filters only Fridays and performs a left join with the `Purchases` table to calculate the total spending.

```sql
WITH RECURSIVE cte AS (
    SELECT '2023-11-01' AS purchase_date
    UNION ALL 
    SELECT purchase_date + INTERVAL 1 DAY AS purchase_date
    FROM cte
    WHERE purchase_date < '2023-11-30'
)

SELECT
    FLOOR(DAY(c.purchase_date) / 7) + 1 AS week_of_month,
    c.purchase_date,
    COALESCE(SUM(p.amount_spend), 0) AS total_amount
FROM cte c
LEFT JOIN Purchases p
    ON c.purchase_date = p.purchase_date
WHERE
    DAYNAME(c.purchase_date) = 'Friday'
GROUP BY 1, 2
ORDER BY 1;
```

### Explanation
1. **CTE (Common Table Expression)**: Generates all dates from `2023-11-01` to `2023-11-30`.
2. **Filter Fridays**: Filters only those dates which fall on a Friday.
3. **Join and Aggregate**: Performs a left join with the `Purchases` table to get the total spending for each Friday, using `COALESCE` to handle missing values.

### Result Format
The result contains `week_of_month`, `purchase_date`, and `total_amount`, ordered by `week_of_month` in ascending order.