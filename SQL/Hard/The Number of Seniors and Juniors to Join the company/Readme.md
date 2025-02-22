# Hiring Candidates

## Table: Candidates

The `Candidates` table contains information about potential employees.

| Column Name  | Type  |
|--------------|-------|
| employee_id  | int   |
| experience   | enum  |
| salary       | int   |

- `employee_id` is the column with unique values for this table.
- `experience` is an ENUM (category) type of values ('Senior', 'Junior').
- Each row of this table indicates the id of a candidate, their monthly salary, and their experience.

## Problem Statement

A company wants to hire new employees with a budget of $70,000. The company's criteria for hiring are:

1. Hire the largest number of seniors.
2. After hiring the maximum number of seniors, use the remaining budget to hire the largest number of juniors.

Write a solution to find the number of seniors and juniors hired under the mentioned criteria.

## Result Format

Return the result table in any order. The result format is as follows:

| experience | accepted_candidates |
|------------|---------------------|
| Senior     | <number>            |
| Junior     | <number>            |

## Examples

### Example 1

**Input:**
Candidates table:
| employee_id | experience | salary |
|-------------|------------|--------|
| 1           | Junior     | 10000  |
| 9           | Junior     | 10000  |
| 2           | Senior     | 20000  |
| 11          | Senior     | 20000  |
| 13          | Senior     | 50000  |
| 4           | Junior     | 40000  |

**Output:**
| experience | accepted_candidates |
|------------|---------------------|
| Senior     | 2                   |
| Junior     | 2                   |

**Explanation:** 
We can hire 2 seniors with IDs (2, 11). Since the budget is $70,000 and the sum of their salaries is $40,000, we still have $30,000 but they are not enough to hire the senior candidate with ID 13. We can hire 2 juniors with IDs (1, 9). Since the remaining budget is $30,000 and the sum of their salaries is $20,000, we still have $10,000 but they are not enough to hire the junior candidate with ID 4.

### Example 2

**Input:**
Candidates table:
| employee_id | experience | salary |
|-------------|------------|--------|
| 1           | Junior     | 10000  |
| 9           | Junior     | 10000  |
| 2           | Senior     | 80000  |
| 11          | Senior     | 80000  |
| 13          | Senior     | 80000  |
| 4           | Junior     | 40000  |

**Output:**
| experience | accepted_candidates |
|------------|---------------------|
| Senior     | 0                   |
| Junior     | 3                   |

**Explanation:** 
We cannot hire any seniors with the current budget as we need at least $80,000 to hire one senior. We can hire all three juniors with the remaining budget.

## SQL Query

```sql
WITH cte AS (
    SELECT
        employee_id,
        experience,
        salary,
        SUM(salary) OVER (PARTITION BY experience ORDER BY salary, employee_id ASC) AS total_salary
    FROM Candidates
)

SELECT
    'Senior' AS experience,
    COUNT(employee_id) AS accepted_candidates
FROM cte
WHERE
    experience = 'Senior'
    AND total_salary < 70000
UNION
SELECT
    'Junior' AS experience,
    COUNT(employee_id) AS accepted_candidates
FROM cte
WHERE
    experience = 'Junior'
    AND total_salary < (SELECT 70000 - COALESCE(MAX(total_salary), 0) FROM cte WHERE experience = 'Senior' AND total_salary < 70000);
```

