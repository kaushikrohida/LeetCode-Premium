# Find Cumulative Salary of an Employee

## Problem Description

Table: Employee

| Column Name | Type |
|-------------|------|
| id          | int  |
| month       | int  |
| salary      | int  |

(id, month) is the primary key for this table.
Each row in the table indicates the salary of an employee in one month during the year 2020.

Write a solution to calculate the cumulative salary summary for every employee in a single unified table.

The cumulative salary summary for an employee can be calculated as follows:

1. For each month that the employee worked, sum up the salaries in that month and the previous two months. This is their 3-month sum for that month. If an employee did not work for the company in previous months, their effective salary for those months is 0.
2. Do not include the 3-month sum for the most recent month that the employee worked for in the summary.
3. Do not include the 3-month sum for any month the employee did not work.

Return the result table ordered by id in ascending order. In case of a tie, order it by month in descending order.

## Example

### Input
Employee table:
```
+----+-------+--------+
| id | month | salary |
+----+-------+--------+
| 1  | 1     | 20     |
| 2  | 1     | 20     |
| 1  | 2     | 30     |
| 2  | 2     | 30     |
| 3  | 2     | 40     |
| 1  | 3     | 40     |
| 3  | 3     | 60     |
| 1  | 4     | 60     |
| 3  | 4     | 70     |
| 1  | 7     | 90     |
| 1  | 8     | 90     |
+----+-------+--------+
```

### Output
```
+----+-------+--------+
| id | month | Salary |
+----+-------+--------+
| 1  | 7     | 90     |
| 1  | 4     | 130    |
| 1  | 3     | 90     |
| 1  | 2     | 50     |
| 1  | 1     | 20     |
| 2  | 1     | 20     |
| 3  | 3     | 100    |
| 3  | 2     | 40     |
+----+-------+--------+
```

## Explanation

Employee '1' has five salary records excluding their most recent month '8':
- 90 for month '7'.
- 60 for month '4'.
- 40 for month '3'.
- 30 for month '2'.
- 20 for month '1'.

So the cumulative salary summary for this employee is:
```
+----+-------+--------+
| id | month | salary |
+----+-------+--------+
| 1  | 7     | 90     |  (90 + 0 + 0)
| 1  | 4     | 130    |  (60 + 40 + 30)
| 1  | 3     | 90     |  (40 + 30 + 20)
| 1  | 2     | 50     |  (30 + 20 + 0)
| 1  | 1     | 20     |  (20 + 0 + 0)
+----+-------+--------+
```
Note that the 3-month sum for month '7' is 90 because they did not work during month '6' or month '5'.

Employee '2' only has one salary record (month '1') excluding their most recent month '2'.
```
+----+-------+--------+
| id | month | salary |
+----+-------+--------+
| 2  | 1     | 20     |  (20 + 0 + 0)
+----+-------+--------+
```

Employee '3' has two salary records excluding their most recent month '4':
- 60 for month '3'.
- 40 for month '2'.

So the cumulative salary summary for this employee is:
```
+----+-------+--------+
| id | month | salary |
+----+-------+--------+
| 3  | 3     | 100    |  (60 + 40 + 0)
| 3  | 2     | 40     |  (40 + 0 + 0)
+----+-------+--------+
```

## Solution

```sql
SELECT
    id,
    month,
    SUM(salary) OVER (
        PARTITION BY id
        ORDER BY month
        RANGE BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS Salary
FROM Employee
WHERE (id, month) NOT IN (
    SELECT id, MAX(month) AS month
    FROM Employee
    GROUP BY id
)
ORDER BY id ASC, month DESC;
```

This solution uses a window function to calculate the cumulative sum of salaries for each employee over a 3-month period. It excludes the most recent month for each employee and orders the results as required.
