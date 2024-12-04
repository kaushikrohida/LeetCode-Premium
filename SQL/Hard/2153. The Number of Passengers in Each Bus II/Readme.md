Table: Buses

+--------------+------+
| Column Name  | Type |
+--------------+------+
| bus_id       | int  |
| arrival_time | int  |
| capacity     | int  |
+--------------+------+
bus_id contains unique values.
Each row of this table contains information about the arrival time of a bus at the LeetCode station and its capacity (the number of empty seats it has).
No two buses will arrive at the same time and all bus capacities will be positive integers.
 

Table: Passengers

+--------------+------+
| Column Name  | Type |
+--------------+------+
| passenger_id | int  |
| arrival_time | int  |
+--------------+------+
passenger_id contains unique values.
Each row of this table contains information about the arrival time of a passenger at the LeetCode station.
 

Buses and passengers arrive at the LeetCode station. If a bus arrives at the station at a time tbus and a passenger arrived at a time tpassenger where tpassenger <= tbus and the passenger did not catch any bus, the passenger will use that bus. In addition, each bus has a capacity. If at the moment the bus arrives at the station there are more passengers waiting than its capacity capacity, only capacity passengers will use the bus.

Write a solution to report the number of users that used each bus.

Return the result table ordered by bus_id in ascending order.

The result format is in the following example.

 

Example 1:

Input: 
Buses table:
+--------+--------------+----------+
| bus_id | arrival_time | capacity |
+--------+--------------+----------+
| 1      | 2            | 1        |
| 2      | 4            | 10       |
| 3      | 7            | 2        |
+--------+--------------+----------+
Passengers table:
+--------------+--------------+
| passenger_id | arrival_time |
+--------------+--------------+
| 11           | 1            |
| 12           | 1            |
| 13           | 5            |
| 14           | 6            |
| 15           | 7            |
+--------------+--------------+
Output: 
+--------+----------------+
| bus_id | passengers_cnt |
+--------+----------------+
| 1      | 1              |
| 2      | 1              |
| 3      | 2              |
+--------+----------------+
Explanation: 
- Passenger 11 arrives at time 1.
- Passenger 12 arrives at time 1.
- Bus 1 arrives at time 2 and collects passenger 11 as it has one empty seat.

- Bus 2 arrives at time 4 and collects passenger 12 as it has ten empty seats.

- Passenger 12 arrives at time 5.
- Passenger 13 arrives at time 6.
- Passenger 14 arrives at time 7.
- Bus 3 arrives at time 7 and collects passengers 12 and 13 as it has two empty seats.


```sql
with recursive cte as (
    select
        b.bus_id,
        b.capacity,
        count(p.passenger_id) as p_count,
        row_number () over (order by b.arrival_time asc) as rnk,
        b.arrival_time
    from Buses b
    Left Join Passengers p
        on b.arrival_time >= p.arrival_time
    group by b.bus_id, b.capacity
),

cte2 as (
    select
        bus_id,
        capacity,
        ifnull(p_count - (lag(p_count) over ()), p_count) as diff, 
        rnk
    from cte
),

cte3 as (
    select
        bus_id,
        least(capacity, diff) as boarded, 
        greatest(0, diff-capacity) as remain,
        rnk
    from cte2
    where rnk = 1
    union all
    select
        c2.bus_id, 
        least(capacity, diff+c3.remain) as boarded,
        greatest(0, diff+c3.remain-capacity) as remain,
        c2.rnk
    from cte2 c2
    Join cte3 c3
        on c2.rnk = c3.rnk + 1
)

select
    bus_id,
    boarded as passengers_cnt
from cte3
order by bus_id
```