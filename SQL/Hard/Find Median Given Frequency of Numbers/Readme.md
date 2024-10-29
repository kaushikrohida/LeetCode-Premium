# Find Median Given Frequency of Numbers

## Problem Description

Table: Numbers

+-------------+------+
| Column Name | Type |
+-------------+------+
| num         | int  |
| frequency   | int  |
+-------------+------+
num is the primary key (column with unique values) for this table.
Each row of this table shows the frequency of a number in the database.

The median is the value separating the higher half from the lower half of a data sample.

Write a solution to report the median of all the numbers in the database after decompressing the Numbers table. Round the median to one decimal point.

## Example

### Input
Numbers table:
+-----+-----------+
| num | frequency |
+-----+-----------+
| 0   | 7         |
| 1   | 1         |
| 2   | 3         |
| 3   | 1         |
+-----+-----------+

### Output
+--------+
| median |
+--------+
| 0.0    |
+--------+

### Explanation
If we decompress the Numbers table, we will get [0, 0, 0, 0, 0, 0, 0, 1, 2, 2, 2, 3], so the median is (0 + 0) / 2 = 0.

## Solution

```sql
WITH cte AS (
    SELECT num
    FROM Numbers
    GROUP BY num, generate_series(1, frequency)
    ORDER BY num
)

SELECT
    ROUND(PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY num), 1) AS median
FROM cte;
```

## Explanation

1. We use a Common Table Expression (CTE) named `cte` to decompress the Numbers table:
   - The `generate_series(1, frequency)` function creates a row for each occurrence of a number based on its frequency.
   - This effectively "unrolls" the compressed data into individual rows.

2. In the main query, we use the `PERCENTILE_CONT` function to calculate the median:
   - `PERCENTILE_CONT(0.5)` calculates the 50th percentile, which is the median.
   - `WITHIN GROUP (ORDER BY num)` specifies the order of the values for calculating the percentile.

3. We use the `ROUND` function to round the result to one decimal place, as required by the problem.

This solution works efficiently even with large datasets, as it doesn't actually materialize the decompressed data, but rather uses SQL's aggregate functions to calculate the median directly from the compressed representation.

## Usage

To use this solution, simply execute the SQL query in a database that supports Common Table Expressions and the `PERCENTILE_CONT` function. Ensure that the `Numbers` table is populated with the appropriate data before running the query.
