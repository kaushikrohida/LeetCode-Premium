# Students Report by Geography

## Problem Description

Table: Student

| Column Name | Type    |
|-------------|---------|
| name        | varchar |
| continent   | varchar |

This table may contain duplicate rows. Each row of this table indicates the name of a student and the continent they came from.

A school has students from Asia, Europe, and America. Write a solution to pivot the continent column in the Student table so that each name is sorted alphabetically and displayed underneath its corresponding continent. The output headers should be America, Asia, and Europe, respectively.

The test cases are generated so that the student number from America is not less than either Asia or Europe.

## Example

### Input
Student table:
| name   | continent |
|--------|-----------|
| Jane   | America   |
| Pascal | Europe    |
| Xi     | Asia      |
| Jack   | America   |

### Output
| America | Asia | Europe |
|---------|------|--------|
| Jack    | Xi   | Pascal |
| Jane    | null | null   |

## Solution

```sql
SELECT
    MAX(CASE WHEN continent = 'America' THEN name END) AS America,
    MAX(CASE WHEN continent = 'Asia' THEN name END) AS Asia,
    MAX(CASE WHEN continent = 'Europe' THEN name END) AS Europe
FROM (
    SELECT
        *,
        ROW_NUMBER() OVER (PARTITION BY continent ORDER BY name) AS rn
    FROM Student
) t
GROUP BY rn
```

## Explanation

1. We start with a subquery that adds a row number for each student within their continent, ordered by name:
   ```sql
   SELECT
       *,
       ROW_NUMBER() OVER (PARTITION BY continent ORDER BY name) AS rn
   FROM Student
   ```

2. In the outer query, we use conditional aggregation with `MAX` and `CASE` statements to pivot the data:
   ```sql
   MAX(CASE WHEN continent = 'America' THEN name END) AS America,
   MAX(CASE WHEN continent = 'Asia' THEN name END) AS Asia,
   MAX(CASE WHEN continent = 'Europe' THEN name END) AS Europe
   ```

3. We group by the row number (`rn`) to ensure that each row in the output represents a single "row" across continents.

This solution works because the `MAX` function, when used with `CASE` statements, effectively pivots the data while preserving the alphabetical order within each continent.

## Follow-up Question

If it is unknown which continent has the most students, you can modify the solution to handle varying numbers of students per continent. Here's an approach:

```sql
WITH RankedStudents AS (
    SELECT
        name,
        continent,
        ROW_NUMBER() OVER (PARTITION BY continent ORDER BY name) AS rn
    FROM Student
),
MaxRows AS (
    SELECT MAX(rn) AS max_rows
    FROM RankedStudents
)
SELECT
    MAX(CASE WHEN continent = 'America' THEN name END) AS America,
    MAX(CASE WHEN continent = 'Asia' THEN name END) AS Asia,
    MAX(CASE WHEN continent = 'Europe' THEN name END) AS Europe
FROM RankedStudents
CROSS JOIN MaxRows
WHERE rn <= max_rows
GROUP BY rn
ORDER BY rn
```

This solution uses a CTE to find the maximum number of students in any continent and ensures that all rows are displayed, even if some continents have fewer students than others.
