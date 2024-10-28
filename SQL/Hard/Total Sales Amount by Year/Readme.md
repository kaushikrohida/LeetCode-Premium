# Total Sales Amount by Year

## Tables

### Table: Product

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| product_id    | int     |
| product_name  | varchar |
+---------------+---------+
product_id is the primary key (column with unique values) for this table.  
product_name is the name of the product.

### Table: Sales

+---------------------+---------+
| Column Name         | Type    |
+---------------------+---------+
| product_id          | int     |
| period_start        | date    |
| period_end          | date    |
| average_daily_sales | int     |
+---------------------+---------+
product_id is the primary key (column with unique values) for this table.  
period_start and period_end indicate the start and end date for the sales period, and both dates are inclusive.  
The average_daily_sales column holds the average daily sales amount of the items for the period.  
The dates of the sales years are between 2018 to 2020.

## Problem Statement

Write a solution to report the total sales amount of each item for each year, with corresponding product_name, product_id, report_year, and total_amount.

Return the result table ordered by product_id and report_year.

### Example

**Input:**

Product table:
+------------+--------------+
| product_id | product_name |
+------------+--------------+
| 1          | LC Phone     |
| 2          | LC T-Shirt   |
| 3          | LC Keychain  |
+------------+--------------+

Sales table:
+------------+--------------+-------------+---------------------+
| product_id | period_start | period_end  | average_daily_sales |
+------------+--------------+-------------+---------------------+
| 1          | 2019-01-25   | 2019-02-28  | 100                 |
| 2          | 2018-12-01   | 2020-01-01  | 10                  |
| 3          | 2019-12-01   | 2020-01-31  | 1                   |
+------------+--------------+-------------+---------------------+

**Output:**
+------------+--------------+-------------+--------------+
| product_id | product_name | report_year | total_amount |
+------------+--------------+-------------+--------------+
| 1          | LC Phone     |    2019     | 3500         |
| 2          | LC T-Shirt   |    2018     | 310          |
| 2          | LC T-Shirt   |    2019     | 3650         |
| 2          | LC T-Shirt   |    2020     | 10           |
| 3          | LC Keychain  |    2019     | 31           |
| 3          | LC Keychain  |    2020     | 31           |
+------------+--------------+-------------+--------------+

**Explanation:**  
- LC Phone was sold for the period of 2019-01-25 to 2019-02-28, and there are 35 days for this period. Total amount 35*100 = 3500.  
- LC T-shirt was sold for the period of 2018-12-01 to 2020-01-01, and there are 31, 365, 1 days for years 2018, 2019 and 2020 respectively.  
- LC Keychain was sold for the period of 2019-12-01 to 2020-01-31, and there are 31, 31 days for years 2019 and 2020 respectively.

## SQL Solution

```sql
WITH RECURSIVE CTE AS (
    SELECT 
        MIN(period_start) as date
    FROM Sales 
    UNION ALL
    SELECT 
        DATE_ADD(date, INTERVAL 1 day)
    FROM CTE
    WHERE date <= ALL (SELECT MAX(period_end) FROM Sales)
)

SELECT
    s.product_id,
    p.product_name,
    DATE_FORMAT(c.date, '%Y') AS report_year,
    SUM(s.average_daily_sales) AS total_amount
FROM Sales s
JOIN Product p
    ON p.product_id = s.product_id
JOIN CTE c
    ON s.period_start <= c.date AND s.period_end >= c.date
GROUP BY 1, 2, 3
ORDER BY 1, 3;