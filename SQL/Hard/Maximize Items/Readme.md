# Inventory Optimization Problem

## Overview

This project solves an inventory optimization problem for a warehouse with a capacity of 500,000 square feet. The warehouse aims to maximize the number of items it can stock, prioritizing prime-eligible items first, followed by non-prime items if there is available space. This solution determines the number of prime and non-prime items that can be stored in the warehouse based on their respective square footage.

### Table Structure
The input table is named `Inventory` and has the following columns:

| Column Name    | Type    | Description                              |
|----------------|---------|------------------------------------------|
| item_id        | int     | Unique identifier for each item          |
| item_type      | varchar | Type of item (e.g., 'prime_eligible', 'not_prime') |
| item_category  | varchar | Category of the item (e.g., 'Watches', 'Clothing') |
| square_footage | decimal | Space required for each item in square feet |

### Problem Statement
- The warehouse has a total capacity of **500,000 square feet**.
- It aims to stock **as many prime items as possible**.
- After maximizing prime items, the **remaining space is used to stock non-prime items**.
- The result should include the **item type** and the **maximum number of items** that can be stored for both categories.
- If there is no space left for non-prime items, the result should output `0` for non-prime items.
- The result should be **ordered by item count in descending order**.

### Example
Given the `Inventory` table:

| item_id | item_type      | item_category | square_footage |
|---------|----------------|---------------|----------------|
| 1374    | prime_eligible | Watches       | 68.00          |
| 4245    | not_prime      | Art           | 26.40          |
| 5743    | prime_eligible | Software      | 325.00         |
| 8543    | not_prime      | Clothing      | 64.50          |
| 2556    | not_prime      | Shoes         | 15.00          |
| 2452    | prime_eligible | Scientific    | 85.00          |
| 3255    | not_prime      | Furniture     | 22.60          |
| 1672    | prime_eligible | Beauty        | 8.50           |
| 4256    | prime_eligible | Furniture     | 55.50          |
| 6325    | prime_eligible | Food          | 13.20          |

The expected output:

| item_type      | item_count |
|----------------|------------|
| prime_eligible | 5400       |
| not_prime      | 8          |

### Solution
The solution to this problem is implemented using SQL, utilizing Common Table Expressions (CTEs) to calculate the maximum number of items that can be stored in the warehouse for both prime and non-prime categories. The approach first calculates the available space occupied by prime items, and then attempts to store as many non-prime items as possible in the remaining space.

### SQL Query
```sql
WITH cte AS (
    SELECT
        item_type,
        SUM(square_footage) * FLOOR(500000 / SUM(square_footage)) AS prime_sq_footage,
        COUNT(item_id) * FLOOR(500000 / SUM(square_footage)) AS item_count
    FROM Inventory
    WHERE item_type = 'prime_eligible'
    GROUP BY item_type
),

cte2 AS (
    SELECT
        item_type,
        CASE
            WHEN (SELECT prime_sq_footage FROM cte) = 500000 THEN 0
            ELSE 500000 - (SELECT prime_sq_footage FROM cte)
        END AS np_sq_footage,
        CASE
            WHEN (SELECT prime_sq_footage FROM cte) = 500000 THEN 0
            ELSE FLOOR((500000 - (SELECT prime_sq_footage FROM cte)) / SUM(square_footage)) * COUNT(item_id)
        END AS item_count
    FROM Inventory
    WHERE item_type = 'not_prime'
)

SELECT item_type, item_count FROM cte
UNION ALL
SELECT item_type, item_count FROM cte2
ORDER BY item_count DESC;
```

### Explanation
- **cte**: Calculates the maximum number of `prime_eligible` items that can be stored based on the available warehouse capacity.
- **cte2**: Calculates the remaining space and determines how many `not_prime` items can be stored.
- The final result is obtained by combining the outputs from both CTEs and ordering the result by `item_count` in descending order.

