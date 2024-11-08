# CEO Subordinates Analysis

## Table: Employees

This project analyzes the hierarchy of employees in a company, focusing on the subordinates of the CEO. Below is the table schema used:

| Column Name   | Type    |
|---------------|---------|
| employee_id   | int     |
| employee_name | varchar |
| manager_id    | int     |
| salary        | int     |

**Primary Key**: `employee_id`

- `employee_id` is the unique identifier for each employee.
- `manager_id` is the `employee_id` of the employee's manager. The CEO has a `NULL` value for `manager_id`.

## Problem Statement

We need to find all subordinates of the CEO, including both direct and indirect reports, along with:

- Their level in the hierarchy (1 for direct reports, 2 for their direct reports, etc.).
- The difference between each subordinate's salary and the CEO's salary.

The result table should be ordered by `hierarchy_level` in ascending order, and then by `subordinate_id` in ascending order.

### Example

**Input**:

| employee_id | employee_name | manager_id | salary  |
|-------------|---------------|------------|---------|
| 1           | Alice         | NULL       | 150000  |
| 2           | Bob           | 1          | 120000  |
| 3           | Charlie       | 1          | 110000  |
| 4           | David         | 2          | 105000  |
| 5           | Eve           | 2          | 100000  |
| 6           | Frank         | 3          | 95000   |
| 7           | Grace         | 3          | 98000   |
| 8           | Helen         | 5          | 90000   |

**Output**:

| subordinate_id | subordinate_name | hierarchy_level | salary_difference |
|----------------|------------------|-----------------|-------------------|
| 2              | Bob              | 1               | -30000            |
| 3              | Charlie          | 1               | -40000            |
| 4              | David            | 2               | -45000            |
| 5              | Eve              | 2               | -50000            |
| 6              | Frank            | 2               | -55000            |
| 7              | Grace            | 2               | -52000            |
| 8              | Helen            | 3               | -60000            |

### Explanation
- **Hierarchy Levels**:
  - Bob and Charlie are direct subordinates of Alice (CEO), so they have a `hierarchy_level` of 1.
  - David and Eve report to Bob, while Frank and Grace report to Charlie, giving them a `hierarchy_level` of 2.
  - Helen reports to Eve, making her a third-level subordinate (`hierarchy_level` of 3).
- **Salary Difference**: Calculated relative to Alice's salary of 150000.
- **Order**: The output is ordered by `hierarchy_level` and then by `subordinate_id`.

## Solution

The solution below uses a Common Table Expression (CTE) to recursively find all subordinates of the CEO, calculating their hierarchy levels and salary differences.

```sql
WITH RECURSIVE cte AS (
    SELECT
        employee_id,
        employee_name,
        0 AS hierarchy_level,
        salary
    FROM Employees
    WHERE manager_id IS NULL
    UNION ALL
    SELECT
        e.employee_id,
        e.employee_name,
        hierarchy_level + 1,
        e.salary - (SELECT salary FROM Employees WHERE manager_id IS NULL)
    FROM Employees e
    JOIN cte c
        ON c.employee_id = e.manager_id
)

SELECT
    employee_id AS subordinate_id,
    employee_name AS subordinate_name,
    hierarchy_level,
    salary AS salary_difference
FROM cte
WHERE 
    hierarchy_level > 0
ORDER BY 3, 1;
```

### Explanation
1. **CTE (Common Table Expression)**: The CTE starts by selecting the CEO, whose `manager_id` is `NULL`. It then recursively finds all subordinates of each employee.
2. **Hierarchy Level Calculation**: The `hierarchy_level` starts from 0 for the CEO and increments for each level down the hierarchy.
3. **Salary Difference**: Calculated as the difference between each subordinate's salary and the CEO's salary.
4. **Final Selection**: The result only includes subordinates (i.e., `hierarchy_level > 0`) and is ordered by `hierarchy_level` and `subordinate_id`.
