Table: students

+-------------+----------+
| Column Name | Type     | 
+-------------+----------+
| student_id  | int      |
| name        | varchar  |
| major       | varchar  |
+-------------+----------+
student_id is the primary key for this table. 
Each row contains the student ID, student name, and their major.
Table: courses

+-------------+-------------------+
| Column Name | Type              |       
+-------------+-------------------+
| course_id   | int               |    
| name        | varchar           |      
| credits     | int               |           
| major       | varchar           |       
| mandatory   | enum              |      
+-------------+-------------------+
course_id is the primary key for this table. 
mandatory is an enum type of ('Yes', 'No').
Each row contains the course ID, course name, credits, major it belongs to, and whether the course is mandatory.
Table: enrollments

+-------------+----------+
| Column Name | Type     | 
+-------------+----------+
| student_id  | int      |
| course_id   | int      |
| semester    | varchar  |
| grade       | varchar  |
| GPA         | decimal  | 
+-------------+----------+
(student_id, course_id, semester) is the primary key (combination of columns with unique values) for this table.
Each row contains the student ID, course ID, semester, and grade received.
Write a solution to find the students who meet the following criteria:

Have taken all mandatory courses and at least two elective courses offered in their major.
Achieved a grade of A in all mandatory courses and at least B in elective courses.
Maintained an average GPA of at least 2.5 across all their courses (including those outside their major).
Return the result table ordered by student_id in ascending order.

 

Example:

Input:

students table:

 +------------+------------------+------------------+
 | student_id | name             | major            |
 +------------+------------------+------------------+
 | 1          | Alice            | Computer Science |
 | 2          | Bob              | Computer Science |
 | 3          | Charlie          | Mathematics      |
 | 4          | David            | Mathematics      |
 +------------+------------------+------------------+
 
courses table:

 +-----------+-------------------+---------+------------------+----------+
 | course_id | name              | credits | major            | mandatory|
 +-----------+-------------------+---------+------------------+----------+
 | 101       | Algorithms        | 3       | Computer Science | yes      |
 | 102       | Data Structures   | 3       | Computer Science | yes      |
 | 103       | Calculus          | 4       | Mathematics      | yes      |
 | 104       | Linear Algebra    | 4       | Mathematics      | yes      |
 | 105       | Machine Learning  | 3       | Computer Science | no       |
 | 106       | Probability       | 3       | Mathematics      | no       |
 | 107       | Operating Systems | 3       | Computer Science | no       |
 | 108       | Statistics        | 3       | Mathematics      | no       |
 +-----------+-------------------+---------+------------------+----------+
 
enrollments table:

 +------------+-----------+-------------+-------+-----+
 | student_id | course_id | semester    | grade | GPA |
 +------------+-----------+-------------+-------+-----+
 | 1          | 101       | Fall 2023   | A     | 4.0 |
 | 1          | 102       | Spring 2023 | A     | 4.0 |
 | 1          | 105       | Spring 2023 | A     | 4.0 |
 | 1          | 107       | Fall 2023   | B     | 3.5 |
 | 2          | 101       | Fall 2023   | A     | 4.0 |
 | 2          | 102       | Spring 2023 | B     | 3.0 |
 | 3          | 103       | Fall 2023   | A     | 4.0 |
 | 3          | 104       | Spring 2023 | A     | 4.0 |
 | 3          | 106       | Spring 2023 | A     | 4.0 |
 | 3          | 108       | Fall 2023   | B     | 3.5 |
 | 4          | 103       | Fall 2023   | B     | 3.0 |
 | 4          | 104       | Spring 2023 | B     | 3.0 |
 +------------+-----------+-------------+-------+-----+
 
Output:

 +------------+
 | student_id |
 +------------+
 | 1          |
 | 3          |
 +------------+
 
Explanation:

Alice (student_id 1) is a Computer Science major and has taken both Algorithms and Data Structures, receiving an A in both. She has also taken Machine Learning and Operating Systems as electives, receiving an A and B respectively.
Bob (student_id 2) is a Computer Science major but did not receive an A in all required courses.
Charlie (student_id 3) is a Mathematics major and has taken both Calculus and Linear Algebra, receiving an A in both. He has also taken Probability and Statistics as electives, receiving an A and B respectively.
David (student_id 4) is a Mathematics major but did not receive an A in all required courses.
Note: Output table is ordered by student_id in ascending order.


```sql
select
    student_id
from (
select
    s.student_id,
    case
        when mandatory = 'yes' and grade = 'A' then 1
        when mandatory = 'yes' and (grade != 'A' or grade is null) then 2 
        else 0
    end as man,
    case
        when mandatory = 'no' and grade in ('A', 'B') then 1
        when mandatory = 'no' and grade not in ('A', 'B') then 2
        else 0
    end as opt
from students s
join courses c
    on s.major = c.major
left join enrollments e
    on s.student_id = e.student_id
    and c.course_id = e.course_id
) a
where
    student_id in (
        select student_id
        from enrollments
        group by 1
        having avg(GPA) >= 2.5
    )
group by 1
having max(man) = 1 and max(opt) = 1 and sum(opt) >= 2
order by 1
```