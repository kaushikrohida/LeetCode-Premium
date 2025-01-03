Table: Student

+---------------------+---------+
| Column Name         | Type    |
+---------------------+---------+
| student_id          | int     |
| student_name        | varchar |
+---------------------+---------+
student_id is the primary key (column with unique values) for this table.
student_name is the name of the student.
 

Table: Exam

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| exam_id       | int     |
| student_id    | int     |
| score         | int     |
+---------------+---------+
(exam_id, student_id) is the primary key (combination of columns with unique values) for this table.
Each row of this table indicates that the student with student_id had a score points in the exam with id exam_id.
 

A quiet student is the one who took at least one exam and did not score the highest or the lowest score.

Write a solution to report the students (student_id, student_name) being quiet in all exams. Do not return the student who has never taken any exam.

Return the result table ordered by student_id.

The result format is in the following example.

 

Example 1:

Input: 
Student table:
+-------------+---------------+
| student_id  | student_name  |
+-------------+---------------+
| 1           | Daniel        |
| 2           | Jade          |
| 3           | Stella        |
| 4           | Jonathan      |
| 5           | Will          |
+-------------+---------------+
Exam table:
+------------+--------------+-----------+
| exam_id    | student_id   | score     |
+------------+--------------+-----------+
| 10         |     1        |    70     |
| 10         |     2        |    80     |
| 10         |     3        |    90     |
| 20         |     1        |    80     |
| 30         |     1        |    70     |
| 30         |     3        |    80     |
| 30         |     4        |    90     |
| 40         |     1        |    60     |
| 40         |     2        |    70     |
| 40         |     4        |    80     |
+------------+--------------+-----------+
Output: 
+-------------+---------------+
| student_id  | student_name  |
+-------------+---------------+
| 2           | Jade          |
+-------------+---------------+
Explanation: 
For exam 1: Student 1 and 3 hold the lowest and high scores respectively.
For exam 2: Student 1 hold both highest and lowest score.
For exam 3 and 4: Studnet 1 and 4 hold the lowest and high scores respectively.
Student 2 and 5 have never got the highest or lowest in any of the exams.
Since student 5 is not taking any exam, he is excluded from the result.
So, we only return the information of Student 2.

```sql
--Approach 01
with cte as (
    select
        e.exam_id,
        s.student_name,
        e.student_id,
        e.score,
        dense_rank () over (partition by e.exam_id order by e.score) rn_asc,
        dense_rank () over (partition by e.exam_id order by e.score desc) rn_desc
    from Exam e
    Join Student s
        on e.student_id = s.student_id
),
cte2 as (
    select
        student_id
    from cte
    where rn_asc = 1 or rn_desc = 1
    group by 1
)

select distinct
    c.student_id,
    c.student_name
from cte c
Left Join cte2 c2
    on c.student_id = c2.student_id
where c2.student_id is null
order by c.student_id
```

```sql
--Approach 02
with cte as (
    select
        e.exam_id,
        s.student_name,
        e.student_id,
        e.score,
        dense_rank () over (partition by e.exam_id order by e.score) rn_asc,
        dense_rank () over (partition by e.exam_id order by e.score desc) rn_desc
    from Exam e
    Join Student s
        on e.student_id = s.student_id
),
cte2 as (
    select distinct
        student_id
    from cte
    where rn_asc = 1
    union all
    select
        student_id
    from cte
    where rn_desc = 1
)

select distinct
    student_id,
    student_name
from cte
where 
    student_id not in (select student_id from cte2)
order by student_id
```