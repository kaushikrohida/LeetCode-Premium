# Tournament Winners

## Tables

### Table: Players

+-------------+-------+
| Column Name | Type  |
+-------------+-------+
| player_id   | int   |
| group_id    | int   |
+-------------+-------+
player_id is the primary key (column with unique values) of this table. Each row of this table indicates the group of each player.

### Table: Matches

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| match_id      | int     |
| first_player  | int     |
| second_player | int     | 
| first_score   | int     |
| second_score  | int     |
+---------------+---------+
match_id is the primary key (column with unique values) of this table. Each row is a record of a match, first_player and second_player contain the player_id of each match. first_score and second_score contain the number of points of the first_player and second_player respectively. You may assume that, in each match, players belong to the same group.

## Problem Statement

The winner in each group is the player who scored the maximum total points within the group. In the case of a tie, the lowest player_id wins.

Write a solution to find the winner in each group.

Return the result table in any order.

## Example

### Input: 
Players table:
+-----------+------------+
| player_id | group_id   |
+-----------+------------+
| 15        | 1          |
| 25        | 1          |
| 30        | 1          |
| 45        | 1          |
| 10        | 2          |
| 35        | 2          |
| 50        | 2          |
| 20        | 3          |
| 40        | 3          |
+-----------+------------+

Matches table:
+------------+--------------+---------------+-------------+--------------+
| match_id   | first_player | second_player | first_score | second_score |
+------------+--------------+---------------+-------------+--------------+
| 1          | 15           | 45            | 3           | 0            |
| 2          | 30           | 25            | 1           | 2            |
| 3          | 30           | 15            | 2           | 0            |
| 4          | 40           | 20            | 5           | 2            |
| 5          | 35           | 50            | 1           | 1            |
+------------+--------------+---------------+-------------+--------------+

### Output: 
+-----------+------------+
| group_id  | player_id  |
+-----------+------------+ 
| 1         | 15         |
| 2         | 35         |
| 3         | 40         |
+-----------+------------+

## SQL Solution

```sql
with cte as (
select
    first_player as player, first_score as score
from Matches
union all
select
    second_player as player, second_score as score 
from Matches
)

select
    group_id as GROUP_ID,
    player as PLAYER_ID
from (
    select
        p.group_id,
        player,
        row_number () over (partition by group_id order by sum(score) desc, player) as rn
    from cte c
    Join Players p
        on c.player = p.player_id
    group by 1, 2
) a
where rn = 1
```