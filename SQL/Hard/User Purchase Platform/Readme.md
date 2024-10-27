# User Purchase Platform

## Problem Description

Write a SQL solution to find the total number of users and the total amount spent using mobile only, desktop only, and both mobile and desktop together for each date.

## Table Structure

Table: Spending

| Column Name | Type    |
|-------------|---------|
| user_id     | int     |
| spend_date  | date    |
| platform    | enum    | 
| amount      | int     |

- (user_id, spend_date, platform) is the primary key of this table.
- The platform column is an ENUM type of ('desktop', 'mobile').
- The table logs the history of user spending on an online shopping website with desktop and mobile applications.

## Example

### Input: 
Spending table:
```
+---------+------------+----------+--------+
| user_id | spend_date | platform | amount |
+---------+------------+----------+--------+
| 1       | 2019-07-01 | mobile   | 100    |
| 1       | 2019-07-01 | desktop  | 100    |
| 2       | 2019-07-01 | mobile   | 100    |
| 2       | 2019-07-02 | mobile   | 100    |
| 3       | 2019-07-01 | desktop  | 100    |
| 3       | 2019-07-02 | desktop  | 100    |
+---------+------------+----------+--------+
```

### Output: 
```
+------------+----------+--------------+-------------+
| spend_date | platform | total_amount | total_users |
+------------+----------+--------------+-------------+
| 2019-07-01 | desktop  | 100          | 1           |
| 2019-07-01 | mobile   | 100          | 1           |
| 2019-07-01 | both     | 200          | 1           |
| 2019-07-02 | desktop  | 100          | 1           |
| 2019-07-02 | mobile   | 100          | 1           |
| 2019-07-02 | both     | 0            | 0           |
+------------+----------+--------------+-------------+
```

### Explanation: 
- On 2019-07-01, user 1 purchased using both desktop and mobile, user 2 purchased using mobile only, and user 3 purchased using desktop only.
- On 2019-07-02, user 2 purchased using mobile only, user 3 purchased using desktop only, and no one purchased using both platforms.

## Solution

```sql
WITH cte AS (
    SELECT
        spend_date,
        user_id,
        CASE
            WHEN COUNT(DISTINCT platform) = 2 THEN 'both'
            ELSE platform
        END AS platform,
        SUM(amount) AS amount
    FROM Spending
    GROUP BY 1, 2
),

cte2 AS (
    SELECT DISTINCT(spend_date), 'desktop' AS platform FROM Spending
    UNION
    SELECT DISTINCT(spend_date), 'mobile' AS platform FROM Spending
    UNION
    SELECT DISTINCT(spend_date), 'both' AS platform FROM Spending
)

SELECT
    c2.spend_date,
    c2.platform,
    COALESCE(SUM(c1.amount), 0) AS total_amount,
    COUNT(DISTINCT c1.user_id) AS total_users
FROM cte2 c2
LEFT JOIN cte c1
    ON c2.spend_date = c1.spend_date
    AND c2.platform = c1.platform
GROUP BY 1, 2
ORDER BY 1, 2

```