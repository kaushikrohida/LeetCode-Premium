# Find Bursty Behavior

## Problem Description

This SQL problem focuses on identifying users who demonstrate bursty behavior in their posting patterns during February 2024. The task involves analyzing the `Posts` table to find users whose posting frequency in any 7-day period is at least twice their average weekly posting frequency for the month.

### Table Structure

The `Posts` table has the following structure:

| Column Name | Type |
|-------------|------|
| post_id     | int  |
| user_id     | int  |
| post_date   | date |

- `post_id` is the primary key of the table.
- Each row contains information about a post, including the user who made it and the date it was posted.

### Task

Write a SQL query to:

1. Calculate the average weekly posting frequency for each user in February 2024.
2. Identify any 7-day period where a user's posting frequency is at least twice their average.
3. Return the results ordered by `user_id` in ascending order.

### Notes

- Only consider dates from February 1 to February 28, 2024.
- Assume February has exactly 4 weeks for calculation purposes.

### Expected Output

The query should return a table with the following columns:

| Column Name      | Description                                               |
|------------------|-----------------------------------------------------------|
| user_id          | The ID of the user                                        |
| max_7day_posts   | Maximum number of posts made in any 7-day period          |
| avg_weekly_posts | Average number of posts per week for the entire month     |

## Example

For a detailed example of input data and expected output, please refer to the problem statement in the SQL file.

## Solution

The solution uses Common Table Expressions (CTEs) and window functions to calculate the required metrics and identify users with bursty behavior. For the complete solution, please check the SQL file in this repository.

with cte as (
    select
        *,
        count(*) over (partition by user_id)/4 as avg_weekly_posts
    from Posts
    where post_date between '2024-02-01' and '2024-02-28'
),

cte2 as (
    select
        *,
        count(post_id) over (partition by user_id order by post_date range between interval 6 day preceding and current row) as max_7day_posts
    from cte
)

select distinct
    user_id,
    max(max_7day_posts) as max_7day_posts,
    avg_weekly_posts
from cte2
where max_7day_posts >= 2*avg_weekly_posts
group by 1
order by 1
