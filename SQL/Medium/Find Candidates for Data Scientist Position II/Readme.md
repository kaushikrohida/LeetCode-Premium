# Find Candidates for Data Scientist Position II

## Problem Description

Leetcode is staffing for multiple data science projects. We need to find the best candidate for each project based on specific criteria.

## Tables

### Candidates

| Column Name  | Type    |
|--------------|---------| 
| candidate_id | int     |
| skill        | varchar |
| proficiency  | int     |

- (candidate_id, skill) is the unique key for this table.
- Each row includes candidate_id, skill, and proficiency level (1-5).

### Projects

| Column Name  | Type    |
|--------------|---------| 
| project_id   | int     |
| skill        | varchar |
| importance   | int     |

- (project_id, skill) is the primary key for this table.
- Each row includes project_id, required skill, and its importance (1-5) for the project.

## Task

Write a solution to find the best candidate for each project based on the following criteria:

1. Candidates must have all the skills required for a project.
2. Calculate a score for each candidate-project pair as follows:
   - Start with 100 points
   - Add 10 points for each skill where proficiency > importance
   - Subtract 5 points for each skill where proficiency < importance
3. Include only the top candidate (highest score) for each project. If there's a tie, choose the candidate with the lower candidate_id.
4. If there is no suitable candidate for a project, do not return that project.
5. Return a result table ordered by project_id in ascending order.

## Example

### Input

Candidates table:

| candidate_id | skill      | proficiency |
|--------------|------------|-------------|
| 101          | Python     | 5           |
| 101          | Tableau    | 3           |
| 101          | PostgreSQL | 4           |
| 101          | TensorFlow | 2           |
| 102          | Python     | 4           |
| 102          | Tableau    | 5           |
| 102          | PostgreSQL | 4           |
| 102          | R          | 4           |
| 103          | Python     | 3           |
| 103          | Tableau    | 5           |
| 103          | PostgreSQL | 5           |
| 103          | Spark      | 4           |

Projects table:

| project_id | skill      | importance |
|------------|------------|------------|
| 501        | Python     | 4          |
| 501        | Tableau    | 3          |
| 501        | PostgreSQL | 5          |
| 502        | Python     | 3          |
| 502        | Tableau    | 4          |
| 502        | R          | 2          |

### Output

| project_id | candidate_id | score |
|------------|--------------|-------|
| 501        | 101          | 105   |
| 502        | 102          | 130   |

### Explanation

- For Project 501, Candidate 101 has the highest score of 105. All other candidates have the same score but Candidate 101 has the lowest candidate_id among them.
- For Project 502, Candidate 102 has the highest score of 130.
- The output table is ordered by project_id in ascending order.

## Solution

```sql
WITH cte AS (
    SELECT DISTINCT 
        c.candidate_id,
        p.project_id,
        SUM(
            CASE
                WHEN proficiency > importance THEN 10
                WHEN proficiency = importance THEN 0
                ELSE -5
            END
        ) + 100 AS score,
        COUNT(*) AS count_skill
    FROM Candidates c
    LEFT JOIN Projects p
        ON c.skill = p.skill
    GROUP BY 1, 2
),

cte2 AS (
    SELECT
        project_id,
        candidate_id,
        score,
        ROW_NUMBER() OVER (PARTITION BY project_id ORDER BY score DESC, candidate_id ASC) AS rn
    FROM cte
    WHERE 
        (project_id, count_skill) IN (SELECT project_id, COUNT(*) FROM Projects GROUP BY 1)
)

SELECT
    project_id,
    candidate_id,
    score
FROM cte2
WHERE
    rn = 1
ORDER BY 1
```

This solution uses Common Table Expressions (CTEs) to break down the problem into manageable steps:

1. The first CTE calculates the score for each candidate-project pair and counts the number of skills matched.
2. The second CTE ranks candidates for each project based on their score and candidate_id.
3. The final SELECT statement retrieves the top candidate for each project, ensuring they have all required skills.

The result is ordered by project_id as required.
