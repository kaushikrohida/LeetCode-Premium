Table: Project

+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| project_id  | int     |
| employee_id | int     |
| workload    | int     |
+-------------+---------+
employee_id is the primary key (column with unique values) of this table.
employee_id is a foreign key (reference column) to Employee table.
Each row of this table indicates that the employee with employee_id is working on the project with project_id and the workload of the project.
Table: Employees

+------------------+---------+
| Column Name      | Type    |
+------------------+---------+
| employee_id      | int     |
| name             | varchar |
| team             | varchar |
+------------------+---------+
employee_id is the primary key (column with unique values) of this table.
Each row of this table contains information about one employee.
Write a solution to find the employees who are allocated to projects with a workload that exceeds the average workload of all employees for their respective teams

Return the result table ordered by employee_id, project_id in ascending order.

The result format is in the following example.

 

Example 1:

Input: 
Project table:
+-------------+-------------+----------+
| project_id  | employee_id | workload |
+-------------+-------------+----------+
| 1           | 1           |  45      |
| 1           | 2           |  90      | 
| 2           | 3           |  12      |
| 2           | 4           |  68      |
+-------------+-------------+----------+
Employees table:
+-------------+--------+------+
| employee_id | name   | team |
+-------------+--------+------+
| 1           | Khaled | A    |
| 2           | Ali    | B    |
| 3           | John   | B    |
| 4           | Doe    | A    |
+-------------+--------+------+
Output: 
+-------------+------------+---------------+------------------+
| employee_id | project_id | employee_name | project_workload |
+-------------+------------+---------------+------------------+  
| 2           | 1          | Ali           | 90               | 
| 4           | 2          | Doe           | 68               | 
+-------------+------------+---------------+------------------+
Explanation: 
- Employee with ID 1 has a project workload of 45 and belongs to Team A, where the average workload is 56.50. Since his project workload does not exceed the team's average workload, he will be excluded.
- Employee with ID 2 has a project workload of 90 and belongs to Team B, where the average workload is 51.00. Since his project workload does exceed the team's average workload, he will be included.
- Employee with ID 3 has a project workload of 12 and belongs to Team B, where the average workload is 51.00. Since his project workload does not exceed the team's average workload, he will be excluded.
- Employee with ID 4 has a project workload of 68 and belongs to Team A, where the average workload is 56.50. Since his project workload does exceed the team's average workload, he will be included.
Result table orderd by employee_id, project_id in ascending order.


```sql
with cte as(
select
    e.team,
    avg(p.workload) as avg_workload
from Project p
join Employees e
    on p.employee_id = e.employee_id
group by 1
)

select
    p.employee_id as EMPLOYEE_ID,
    p.project_id as PROJECT_ID,
    e.name as EMPLOYEE_NAME,
    p.workload as PROJECT_WORKLOAD
from Project p
join Employees e
    on p.employee_id = e.employee_id
join cte c
    on e.team = c.team
where p.workload > c.avg_workload
order by 1 asc, 2 asc
```