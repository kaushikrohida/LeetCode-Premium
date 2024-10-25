# Medium Employee Salary

## Problem Description

Table: Employee

+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| id           | int     |
| company      | varchar |
| salary       | int     |
+--------------+---------+

- `id` is the primary key (column with unique values) for this table.
- Each row of this table indicates the company and the salary of one employee.

Write a solution to find the rows that contain the median salary of each company. While calculating the median, when you sort the salaries of the company, break the ties by id.

Return the result table in any order.

## Example

### Input
Employee table:
```
+----+---------+--------+
| id | company | salary |
+----+---------+--------+
| 1  | A       | 2341   |
| 2  | A       | 341    |
| 3  | A       | 15     |
| 4  | A       | 15314  |
| 5  | A       | 451    |
| 6  | A       | 513    |
| 7  | B       | 15     |
| 8  | B       | 13     |
| 9  | B       | 1154   |
| 10 | B       | 1345   |
| 11 | B       | 1221   |
| 12 | B       | 234    |
| 13 | C       | 2345   |
| 14 | C       | 2645   |
| 15 | C       | 2645   |
| 16 | C       | 2652   |
| 17 | C       | 65     |
+----+---------+--------+
```

### Output
```
+----+---------+--------+
| id | company | salary |
+----+---------+--------+
| 5  | A       | 451    |
| 6  | A       | 513    |
| 12 | B       | 234    |
| 9  | B       | 1154   |
| 14 | C       | 2645   |
+----+---------+--------+
```

### Explanation
For company A, the median salaries are 451 and 513.
For company B, the median salaries are 234 and 1154.
For company C, the median salary is 2645.

## Solution

```sql
SELECT
    id,
    company,
    salary
FROM (
    SELECT
        *,
        ROW_NUMBER() OVER (PARTITION BY company ORDER BY salary, id) AS rn_asc,
        ROW_NUMBER() OVER (PARTITION BY company ORDER BY salary DESC, id DESC) AS rn_desc
    FROM Employee
) a
WHERE rn_asc BETWEEN rn_desc - 1 AND rn_desc + 1
ORDER BY company, salary;
```

This solution uses window functions to calculate the median salary for each company. It works by:

1. Assigning ascending and descending row numbers to each salary within each company.
2. Selecting rows where the ascending row number is within 1 of the descending row number, which identifies the median value(s).
3. Ordering the result by company and salary.

## Follow-up Question

Could you solve it without using any built-in or window functions?
