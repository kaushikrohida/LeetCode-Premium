# Find Longest Calls

## Problem Description

You are given two tables: `Contacts` and `Calls`. The task is to find the three longest incoming and outgoing calls, format their duration as HH:MM:SS, and return the result ordered by type, duration, and first name in descending order.

## Tables

### Contacts

| Column Name | Type    |
|-------------|---------|
| id          | int     |
| first_name  | varchar |
| last_name   | varchar |

- `id` is the primary key of this table.
- `id` is a foreign key to the Calls table.
- Each row contains id, first_name, and last_name.

### Calls

| Column Name | Type |
|-------------|------|
| contact_id  | int  |
| type        | enum |
| duration    | int  |

- `(contact_id, type, duration)` is the primary key of this table.
- `type` is an ENUM of ('incoming', 'outgoing').
- Each row contains information about calls: contact_id, type, and duration in seconds.

## Example

### Input

Contacts table:

| id | first_name | last_name |
|----|------------|-----------|
| 1  | John       | Doe       |
| 2  | Jane       | Smith     |
| 3  | Alice      | Johnson   |
| 4  | Michael    | Brown     |
| 5  | Emily      | Davis     |

Calls table:

| contact_id | type     | duration |
|------------|----------|----------|
| 1          | incoming | 120      |
| 1          | outgoing | 180      |
| 2          | incoming | 300      |
| 2          | outgoing | 240      |
| 3          | incoming | 150      |
| 3          | outgoing | 360      |
| 4          | incoming | 420      |
| 4          | outgoing | 200      |
| 5          | incoming | 180      |
| 5          | outgoing | 280      |

### Output

| first_name | type     | duration_formatted |
|------------|----------|---------------------|
| Alice      | outgoing | 00:06:00            |
| Emily      | outgoing | 00:04:40            |
| Jane       | outgoing | 00:04:00            |
| Michael    | incoming | 00:07:00            |
| Jane       | incoming | 00:05:00            |
| Emily      | incoming | 00:03:00            |

## Explanation

- Alice had an outgoing call lasting 6 minutes.
- Emily had an outgoing call lasting 4 minutes and 40 seconds.
- Jane had an outgoing call lasting 4 minutes.
- Michael had an incoming call lasting 7 minutes.
- Jane had an incoming call lasting 5 minutes.
- Emily had an incoming call lasting 3 minutes.

Note: The output table is sorted by type, duration, and first_name in descending order.

## Solution

```sql
SELECT
    c.first_name,
    a.type,
    DATE_FORMAT(SEC_TO_TIME(a.duration), '%H:%i:%s') AS duration_formatted
FROM (
    SELECT
        contact_id,
        type,
        duration,
        ROW_NUMBER() OVER (PARTITION BY type ORDER BY duration DESC) AS rn
    FROM Calls
) a
JOIN Contacts c
    ON a.contact_id = c.id
WHERE
    rn <= 3
ORDER BY
    a.type DESC, duration DESC, c.first_name
```

This solution uses a subquery with `ROW_NUMBER()` to rank the calls by duration within each type (incoming/outgoing). It then joins the result with the Contacts table, selects the top 3 for each type, and formats the duration as required.
