# Game Play Analysis V

## Problem Description

Table: Activity

+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| player_id    | int     |
| device_id    | int     |
| event_date   | date    |
| games_played | int     |
+--------------+---------+
(player_id, event_date) is the primary key of this table.
This table shows the activity of players of some games.
Each row is a record of a player who logged in and played a number of games (possibly 0) before logging out on someday using some device.

The install date of a player is the first login day of that player.

We define day one retention of some date x to be the number of players whose install date is x and they logged back in on the day right after x, divided by the number of players whose install date is x, rounded to 2 decimal places.

Write a solution to report for each install date, the number of players that installed the game on that day, and the day one retention.

Return the result table in any order.

## Example

### Input
Activity table:
+-----------+-----------+------------+--------------+
| player_id | device_id | event_date | games_played |
+-----------+-----------+------------+--------------+
| 1         | 2         | 2016-03-01 | 5            |
| 1         | 2         | 2016-03-02 | 6            |
| 2         | 3         | 2017-06-25 | 1            |
| 3         | 1         | 2016-03-01 | 0            |
| 3         | 4         | 2016-07-03 | 5            |
+-----------+-----------+------------+--------------+

### Output
+------------+----------+----------------+
| install_dt | installs | Day1_retention |
+------------+----------+----------------+
| 2016-03-01 | 2        | 0.50           |
| 2017-06-25 | 1        | 0.00           |
+------------+----------+----------------+

### Explanation
- Player 1 and 3 installed the game on 2016-03-01 but only player 1 logged back in on 2016-03-02 so the day 1 retention of 2016-03-01 is 1 / 2 = 0.50
- Player 2 installed the game on 2017-06-25 but didn't log back in on 2017-06-26 so the day 1 retention of 2017-06-25 is 0 / 1 = 0.00

## Solution

```sql
WITH cte AS (
    SELECT
        *,
        MIN(event_date) OVER (PARTITION BY player_id ORDER BY event_date) AS install_dt
    FROM Activity a1
)

SELECT
    install_dt,
    COUNT(DISTINCT player_id) AS installs,
    ROUND(SUM(event_date = install_dt + INTERVAL 1 DAY) / COUNT(DISTINCT player_id), 2) AS Day1_retention
FROM cte
GROUP BY 1
```

### Explanation:

1. We use a Common Table Expression (CTE) to find the install date for each player.
2. In the main query, we group by the install date and calculate:
   - The number of installs (distinct players)
   - The day 1 retention rate
3. The retention rate is calculated by:
   - Counting players who logged in the day after their install date
   - Dividing by the total number of players who installed on that date
   - Rounding to 2 decimal places

This solution efficiently handles the requirements of the problem, providing the install date, number of installs, and day 1 retention rate for each install date.
