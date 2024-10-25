# Trips and Users

## Problem Description

We have two tables: `Trips` and `Users`. We need to find the cancellation rate of requests with unbanned users (both client and driver must not be banned) each day between "2013-10-01" and "2013-10-03". The cancellation rate should be rounded to two decimal points.

## Tables

### Trips

| Column Name | Type    |
|-------------|---------|
| id          | int     |
| client_id   | int     |
| driver_id   | int     |
| city_id     | int     |
| status      | enum    |
| request_at  | varchar |

- `id` is the primary key for this table.
- `client_id` and `driver_id` are foreign keys to the `users_id` in the Users table.
- `status` is an ENUM type of ('completed', 'cancelled_by_driver', 'cancelled_by_client').

### Users

| Column Name | Type |
|-------------|------|
| users_id    | int  |
| banned      | enum |
| role        | enum |

- `users_id` is the primary key for this table.
- `role` is an ENUM type of ('client', 'driver', 'partner').
- `banned` is an ENUM type of ('Yes', 'No').

## Example

### Input

Trips table:
```
+----+-----------+-----------+---------+---------------------+------------+
| id | client_id | driver_id | city_id | status              | request_at |
+----+-----------+-----------+---------+---------------------+------------+
| 1  | 1         | 10        | 1       | completed           | 2013-10-01 |
| 2  | 2         | 11        | 1       | cancelled_by_driver | 2013-10-01 |
| 3  | 3         | 12        | 6       | completed           | 2013-10-01 |
| 4  | 4         | 13        | 6       | cancelled_by_client | 2013-10-01 |
| 5  | 1         | 10        | 1       | completed           | 2013-10-02 |
| 6  | 2         | 11        | 6       | completed           | 2013-10-02 |
| 7  | 3         | 12        | 6       | completed           | 2013-10-02 |
| 8  | 2         | 12        | 12      | completed           | 2013-10-03 |
| 9  | 3         | 10        | 12      | completed           | 2013-10-03 |
| 10 | 4         | 13        | 12      | cancelled_by_driver | 2013-10-03 |
+----+-----------+-----------+---------+---------------------+------------+
```

Users table:
```
+----------+--------+--------+
| users_id | banned | role   |
+----------+--------+--------+
| 1        | No     | client |
| 2        | Yes    | client |
| 3        | No     | client |
| 4        | No     | client |
| 10       | No     | driver |
| 11       | No     | driver |
| 12       | No     | driver |
| 13       | No     | driver |
+----------+--------+--------+
```

### Output

```
+------------+-------------------+
| Day        | Cancellation Rate |
+------------+-------------------+
| 2013-10-01 | 0.33              |
| 2013-10-02 | 0.00              |
| 2013-10-03 | 0.50              |
+------------+-------------------+
```

## Solution

```sql
SELECT
    t.request_at AS 'Day',
    ROUND(COUNT(CASE WHEN t.status LIKE 'cancelled%' THEN 1 ELSE NULL END) / COUNT(*), 2) AS 'Cancellation Rate'
FROM Trips t
JOIN Users u1
    ON t.client_id = u1.users_id AND u1.role = 'client' AND u1.banned = 'No'
JOIN Users u2
    ON t.driver_id = u2.users_id AND u2.role = 'driver' AND u2.banned = 'No'
WHERE 
    t.request_at BETWEEN '2013-10-01' AND '2013-10-03'
GROUP BY 1
ORDER BY 1
```

## Explanation

1. We join the `Trips` table with the `Users` table twice:
   - Once for the client (u1)
   - Once for the driver (u2)
2. We filter out banned users in the JOIN conditions.
3. We count the number of cancelled trips (where status starts with 'cancelled') and divide it by the total number of trips for each day.
4. We use the ROUND function to round the result to 2 decimal places.
5. We filter the dates to be between '2013-10-01' and '2013-10-03'.
6. We group and order the results by the request date.

This solution efficiently calculates the cancellation rate for unbanned users within the specified date range.
