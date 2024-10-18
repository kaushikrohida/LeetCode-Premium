# Find Top Performing Driver

## Problem Description

Uber is analyzing drivers based on their trips. Write a solution to find the top-performing driver for each fuel type based on the following criteria:

1. A driver's performance is calculated as the average rating across all their trips. Average rating should be rounded to 2 decimal places.
2. If two drivers have the same average rating, the driver with the longer total distance traveled should be ranked higher.
3. If there is still a tie, choose the driver with the fewest accidents.
4. Return the result table ordered by fuel_type in ascending order.

## Tables

### Drivers

+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| driver_id    | int     |
| name         | varchar |
| age          | int     |
| experience   | int     |
| accidents    | int     |
+--------------+---------+

- (driver_id) is the unique key for this table.
- Each row includes a driver's ID, their name, age, years of driving experience, and the number of accidents they've had.

### Vehicles

+--------------+---------+
| vehicle_id   | int     |
| driver_id    | int     |
| model        | varchar |
| fuel_type    | varchar |
| mileage      | int     |
+--------------+---------+

- (vehicle_id, driver_id, fuel_type) is the unique key for this table.
- Each row includes the vehicle's ID, the driver who operates it, the model, fuel type, and mileage.

### Trips

+--------------+---------+
| trip_id      | int     |
| vehicle_id   | int     |
| distance     | int     |
| duration     | int     |
| rating       | int     |
+--------------+---------+

- (trip_id) is the unique key for this table.
- Each row includes a trip's ID, the vehicle used, the distance covered (in miles), the trip duration (in minutes), and the passenger's rating (1-5).

## Example

### Input

Drivers table:
+-----------+----------+-----+------------+-----------+
| driver_id | name     | age | experience | accidents |
+-----------+----------+-----+------------+-----------+
| 1         | Alice    | 34  | 10         | 1         |
| 2         | Bob      | 45  | 20         | 3         |
| 3         | Charlie  | 28  | 5          | 0         |
+-----------+----------+-----+------------+-----------+

Vehicles table:
+------------+-----------+---------+-----------+---------+
| vehicle_id | driver_id | model   | fuel_type | mileage |
+------------+-----------+---------+-----------+---------+
| 100        | 1         | Sedan   | Gasoline  | 20000   |
| 101        | 2         | SUV     | Electric  | 30000   |
| 102        | 3         | Coupe   | Gasoline  | 15000   |
+------------+-----------+---------+-----------+---------+

Trips table:
+---------+------------+----------+----------+--------+
| trip_id | vehicle_id | distance | duration | rating |
+---------+------------+----------+----------+--------+
| 201     | 100        | 50       | 30       | 5      |
| 202     | 100        | 30       | 20       | 4      |
| 203     | 101        | 100      | 60       | 4      |
| 204     | 101        | 80       | 50       | 5      |
| 205     | 102        | 40       | 30       | 5      |
| 206     | 102        | 60       | 40       | 5      |
+---------+------------+----------+----------+--------+

### Output

+-----------+-----------+--------+----------+
| fuel_type | driver_id | rating | distance |
+-----------+-----------+--------+----------+
| Electric  | 2         | 4.50   | 180      |
| Gasoline  | 3         | 5.00   | 100      |
+-----------+-----------+--------+----------+

### Explanation

- For fuel type Gasoline, both Alice (Driver 1) and Charlie (Driver 3) have trips. Charlie has an average rating of 5.0, while Alice has 4.5. Therefore, Charlie is selected.
- For fuel type Electric, Bob (Driver 2) is the only driver with an average rating of 4.5, so he is selected.
- The output table is ordered by fuel_type in ascending order.

## Solution

```sql
with cte as(
select
    v.fuel_type,
    d.driver_id,
    round(avg(t.rating), 2) as avg_rating,
    sum(t.distance) as total_distance,
    sum(d.accidents) as total_accidents,
    row_number () over (partition by v.fuel_type order by avg(t.rating) desc, sum(t.distance) desc, sum(d.accidents) asc) rn
from Trips t
Left Join Vehicles v
    on t.vehicle_id = v.vehicle_id
Left Join Drivers d
    on v.driver_id = d.driver_id
group by 1, 2
)

select
    fuel_type,
    driver_id,
    avg_rating as rating,
    total_distance as distance
from cte
where
    rn = 1
order by 1
```

This solution uses a Common Table Expression (CTE) to calculate the average rating, total distance, and total accidents for each driver, grouped by fuel type. It then uses the `ROW_NUMBER()` function to rank the drivers within each fuel type based on the given criteria. Finally, it selects the top-performing driver for each fuel type by filtering for `rn = 1` and orders the result by fuel type.
