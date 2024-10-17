# Calculate Parking Fees and Duration

## Problem Description

Given a table `ParkingTransactions`, write a solution to find the total parking fee paid by each car across all parking lots, and the average hourly fee (rounded to 2 decimal places) paid by each car. Also, find the parking lot where each car spent the most total time.

Return the result table ordered by car_id in ascending order.

Note: Test cases are generated in such a way that an individual car cannot be in multiple parking lots at the same time.

## Table Structure

Table: ParkingTransactions

| Column Name | Type     |
|-------------|----------|
| lot_id      | int      |
| car_id      | int      |
| entry_time  | datetime |
| exit_time   | datetime |
| fee_paid    | decimal  |

(lot_id, car_id, entry_time) is the primary key for this table.

## Example

### Input:

ParkingTransactions table:

| lot_id | car_id | entry_time          | exit_time           | fee_paid |
|--------|--------|---------------------|---------------------|----------|
| 1      | 1001   | 2023-06-01 08:00:00 | 2023-06-01 10:30:00 | 5.00     |
| 1      | 1001   | 2023-06-02 11:00:00 | 2023-06-02 12:45:00 | 3.00     |
| 2      | 1001   | 2023-06-01 10:45:00 | 2023-06-01 12:00:00 | 6.00     |
| 2      | 1002   | 2023-06-01 09:00:00 | 2023-06-01 11:30:00 | 4.00     |
| 3      | 1001   | 2023-06-03 07:00:00 | 2023-06-03 09:00:00 | 4.00     |
| 3      | 1002   | 2023-06-02 12:00:00 | 2023-06-02 14:00:00 | 2.00     |

### Output:

| car_id | total_fee_paid | avg_hourly_fee | most_time_lot |
|--------|----------------|----------------|---------------|
| 1001   | 18.00          | 2.40           | 1             |
| 1002   | 6.00           | 1.33           | 2             |

### Explanation:

For car ID 1001:
- From 2023-06-01 08:00:00 to 2023-06-01 10:30:00 in lot 1: 2.5 hours, fee 5.00
- From 2023-06-02 11:00:00 to 2023-06-02 12:45:00 in lot 1: 1.75 hours, fee 3.00
- From 2023-06-01 10:45:00 to 2023-06-01 12:00:00 in lot 2: 1.25 hours, fee 6.00
- From 2023-06-03 07:00:00 to 2023-06-03 09:00:00 in lot 3: 2 hours, fee 4.00
Total fee paid: 18.00, total hours: 7.5, average hourly fee: 2.40, most time spent in lot 1: 4.25 hours.

For car ID 1002:
- From 2023-06-01 09:00:00 to 2023-06-01 11:30:00 in lot 2: 2.5 hours, fee 4.00
- From 2023-06-02 12:00:00 to 2023-06-02 14:00:00 in lot 3: 2 hours, fee 2.00
Total fee paid: 6.00, total hours: 4.5, average hourly fee: 1.33, most time spent in lot 2: 2.5 hours.

Note: Output table is ordered by car_id in ascending order.

## Solution

```sql
# Write your MySQL query statement below
with cte as (
select
    car_id,
    sum(fee_paid)as total_fee_paid,
    round(sum(fee_paid) / sum(timestampdiff(second, entry_time, exit_time) / 3600), 2) as avg_hourly_fee
from ParkingTransactions
group by 1
),

cte2 as (
select
    car_id, 
    lot_id,
    sum(timestampdiff(second, entry_time, exit_time) / 3600),
    dense_rank () over (partition by car_id order by sum(timestampdiff(second, entry_time, exit_time) / 3600) desc) dr
from ParkingTransactions
group by 1, 2
)

select
    c1.car_id,
    c1.total_fee_paid,
    c1.avg_hourly_fee,
    c2.lot_id as most_time_lot
from cte c1
Join cte2 c2
    on c1.car_id = c2.car_id
where c2.dr = 1
order by 1
```

This solution uses Common Table Expressions (CTEs) to calculate the required metrics and then joins the results to produce the final output.
