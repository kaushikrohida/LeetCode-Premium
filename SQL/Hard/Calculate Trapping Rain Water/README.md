# Calculate Trapping Water

## Description
This project provides a solution to calculate the amount of rainwater that can be trapped between bars in a landscape, represented by a table of heights. Each bar has a width of 1 unit, and the solution utilizes SQL to compute the total trapped water based on the heights provided.

## Table Structure
The `Heights` table contains the following columns:

+-------------+------+
| Column Name | Type |
+-------------+------+
| id          | int  |
| height      | int  |
+-------------+------+

- `id`: The primary key, which is guaranteed to be in sequential order.
- `height`: The height of the bar at the given id.

## Problem Statement
Write a SQL query to calculate the total amount of rainwater that can be trapped between the bars. The result should be returned in any order.

## Example
### Input
Heights table:
+-----+--------+
| id  | height |
+-----+--------+
| 1   | 0      |
| 2   | 1      |
| 3   | 0      |
| 4   | 2      |
| 5   | 1      |
| 6   | 0      |
| 7   | 1      |
| 8   | 3      |
| 9   | 2      |
| 10  | 1      |
| 11  | 2      |
| 12  | 1      |
+-----+--------+

### Output
+---------------------+
| total_trapped_water | 
+---------------------+
| 6                   | 
+---------------------+

### Explanation
The elevation map is represented graphically with the x-axis denoting the id and the y-axis representing the heights. In this scenario, 6 units of rainwater are trapped.

## SQL Solution
The following SQL query calculates the total trapped water:

```sql
with cte as(
select
    id, 
    height,
    max(height) over (order by id) as left_bar,
    max(height) over (order by id desc) as right_bar
from Heights
order by id
)

select
    sum(least(left_bar, right_bar) - height) as total_trapped_water
from cte
```
