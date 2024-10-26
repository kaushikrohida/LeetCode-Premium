# Average Salary: Departments VS Company

## Problem Description

Compare the average salary of employees in each department to the company's average salary.

## Schema

### Table: Salary

| Column Name | Type |
|-------------|------|
| id          | int  |
| employee_id | int  |
| amount      | int  |
| pay_date    | date |

- In SQL, id is the primary key column for this table.
- Each row of this table indicates the salary of an employee in one month.
- employee_id is a foreign key (reference column) from the Employee table.

### Table: Employee

| Column Name   | Type |
|---------------|------|
| employee_id   | int  |
| department_id | int  |

- In SQL, employee_id is the primary key column for this table.
- Each row of this table indicates the department of an employee.

## Task

Find the comparison result (higher/lower/same) of the average salary of employees in a department to the company's average salary.

Return the result table in any order.

## Example

### Input

Salary table:

| id | employee_id | amount | pay_date   |
|----|-------------|--------|------------|
| 1  | 1           | 9000   | 2017/03/31 |
| 2  | 2           | 6000   | 2017/03/31 |
| 3  | 3           | 10000  | 2017/03/31 |
| 4  | 1           | 7000   | 2017/02/28 |
| 5  | 2           | 6000   | 2017/02/28 |
| 6  | 3           | 8000   | 2017/02/28 |

Employee table:

| employee_id | department_id |
|-------------|---------------|
| 1           | 1             |
| 2           | 2             |
| 3           | 2             |

### Output

| pay_month | department_id | comparison |
|-----------|---------------|------------|
| 2017-02   | 1             | same       |
| 2017-03   | 1             | higher     |
| 2017-02   | 2             | same       |
| 2017-03   | 2             | lower      |

### Explanation

In March, the company's average salary is (9000+6000+10000)/3 = 8333.33...
- The average salary for department '1' is 9000, which is the salary of employee_id '1' since there is only one employee in this department. So the comparison result is 'higher' since 9000 > 8333.33 obviously.
- The average salary of department '2' is (6000 + 10000)/2 = 8000, which is the average of employee_id '2' and '3'. So the comparison result is 'lower' since 8000 < 8333.33.

With the same formula for the average salary comparison in February, the result is 'same' since both department '1' and '2' have the same average salary as the company, which is 7000.

## Solution

```sql
WITH cte AS (
    SELECT
        DATE_FORMAT(pay_date, "%Y-%m") AS pay_month,
        e.department_id,
        SUM(s.amount)/COUNT(*) AS avg_dep_amount
    FROM Salary s
    JOIN Employee e ON s.employee_id = e.employee_id
    GROUP BY 1, 2
    ORDER BY 1, 2
),
cte2 AS (
    SELECT
        DATE_FORMAT(pay_date, "%Y-%m") AS pay_month,
        AVG(amount) AS avg_comp_amount
    FROM Salary
    GROUP BY 1
)

SELECT
    c1.pay_month,
    c1.department_id,
    CASE
        WHEN c1.avg_dep_amount > c2.avg_comp_amount THEN "higher"
        WHEN c1.avg_dep_amount < c2.avg_comp_amount THEN "lower"
        ELSE "same"
    END AS comparison
FROM cte c1
LEFT JOIN cte2 c2 ON c1.pay_month = c2.pay_month
ORDER BY 2
```

This solution uses Common Table Expressions (CTEs) to calculate the average salary for each department and the company as a whole. It then compares these averages to determine whether each department's average is higher, lower, or the same as the company average.
