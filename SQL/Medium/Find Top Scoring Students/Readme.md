# Find Top Scoring Students

## Problem Description

Write a solution to find the students who have taken all courses offered in their major and have achieved a grade of A in all these courses. Return the result table ordered by student_id in ascending order.

## Tables

### students

| Column Name | Type    |
|-------------|---------|
| student_id  | int     |
| name        | varchar |
| major       | varchar |

student_id is the primary key for this table.
Each row of this table contains the student ID, student name, and their major.

### courses

| Column Name | Type    |
|-------------|---------|
| course_id   | int     |
| name        | varchar |
| credits     | int     |
| major       | varchar |

course_id is the primary key for this table.
Each row of this table contains the course ID, course name, the number of credits for the course, and the major it belongs to.

### enrollments

| Column Name | Type    |
|-------------|---------|
| student_id  | int     |
| course_id   | int     |
| semester    | varchar |
| grade       | varchar |

(student_id, course_id, semester) is the primary key for this table.
Each row of this table contains the student ID, course ID, semester, and grade received.

## Example

### Input

students table:

| student_id | name    | major            |
|------------|---------|------------------|
| 1          | Alice   | Computer Science |
| 2          | Bob     | Computer Science |
| 3          | Charlie | Mathematics      |
| 4          | David   | Mathematics      |

courses table:

| course_id | name            | credits | major            |
|-----------|-----------------|---------|------------------|
| 101       | Algorithms      | 3       | Computer Science |
| 102       | Data Structures | 3       | Computer Science |
| 103       | Calculus        | 4       | Mathematics      |
| 104       | Linear Algebra  | 4       | Mathematics      |

enrollments table:

| student_id | course_id | semester  | grade |
|------------|-----------|-----------|-------|
| 1          | 101       | Fall 2023 | A     |
| 1          | 102       | Fall 2023 | A     |
| 2          | 101       | Fall 2023 | B     |
| 2          | 102       | Fall 2023 | A     |
| 3          | 103       | Fall 2023 | A     |
| 3          | 104       | Fall 2023 | A     |
| 4          | 103       | Fall 2023 | A     |
| 4          | 104       | Fall 2023 | B     |

### Output

| student_id |
|------------|
| 1          |
| 3          |

### Explanation

- Alice (student_id 1) is a Computer Science major and has taken both "Algorithms" and "Data Structures", receiving an 'A' in both.
- Bob (student_id 2) is a Computer Science major but did not receive an 'A' in all required courses.
- Charlie (student_id 3) is a Mathematics major and has taken both "Calculus" and "Linear Algebra", receiving an 'A' in both.
- David (student_id 4) is a Mathematics major but did not receive an 'A' in all required courses.

Note: Output table is ordered by student_id in ascending order.

## Solution

```sql
select
    s.student_id
from students s
Join courses c
    on s.major = c.major
Left Join enrollments e
    on s.student_id = e.student_id and c.course_id = e.course_id and e.grade = 'A'
group by 1
having count(distinct c.course_id) = count(distinct e.course_id)
order by 1
```

This solution joins the students table with the courses table based on the major, and then left joins with the enrollments table. It groups the results by student_id and uses a HAVING clause to ensure that the number of distinct courses in the student's major matches the number of distinct courses they've taken with an 'A' grade. The results are ordered by student_id in ascending order.
