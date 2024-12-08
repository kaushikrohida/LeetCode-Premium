You are tasked with managing project budgets at a company. Each project has a fixed budget, and multiple employees work on these projects. The company's payroll is based on annual salaries, and each employee works for a specific duration on a project.

 

Over budget on a project is defined when the salaries (allocated on per day basis as per project duration) exceed the budget of the project. For example, if Ankit and Rohit both combined income make 200K and work on a project of a budget of 50K that takes half a year, then the project is over budget given 0.5 * 200K = 100K > 50K.

 

Write a query to forecast the budget for all projects and return a label of "overbudget" if it is over budget and "within budget" otherwise. Order the result by project title.

 Note: Assume that employees only work on one project at a time.

 

Table: employees 
+-------------+----------+
| COLUMN_NAME | DATA_TYPE|
+-------------+----------+
| id          | int      |
| name        | varchar  |
| salary      | int      |
+-------------+----------+
Table: projects 
+-------------+----------+
| COLUMN_NAME | DATA_TYPE|
+-------------+----------+
| id          | int      |
| title       | varchar  |
| start_date  | date     |
| end_date    | date     |
| budget      | int      |
+-------------+----------+
Table: project_employees 
+-------------+----------+
| COLUMN_NAME | DATA_TYPE|
+-------------+----------+
| project_id  | int      |
| employee_id | int      |
+-------------+----------+


```sql
with cte as (
  select
      p.title as project_title,
      p.budget,
      sum(e.salary * (datediff(p.end_date, p.start_date) / 365.0)) as total_salary_cost
  from Projects p
  join Project_Employees pe
      on p.id = pe.project_id
  join Employees e
      on e.id = pe.employee_id
  group by 
      p.id,
      p.title,
      p.budget
)

select
	project_title as title,
    budget,
    case
    	when total_salary_cost > budget then 'overbudget'
        else 'within budget'
    end as label
from cte
order by 1
```