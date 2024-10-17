# Calculate Product Final Price

## Problem Description

Given two tables, `Products` and `Discounts`, write a SQL query to find the final price of each product after applying the category discount. If a product's category has no associated discount, its price remains unchanged.

### Table: Products

| Column Name | Type    |
|-------------|---------|
| product_id  | int     |
| category    | varchar |
| price       | decimal |

- `product_id` is the primary key for this table.
- Each row includes the product's ID, its category, and its price.

### Table: Discounts

| Column Name | Type    |
|-------------|---------|
| category    | varchar |
| discount    | int     |

- `category` is the primary key for this table.
- Each row contains a product category and the percentage discount applied to that category (values range from 0 to 100).

## Task

Write a SQL query to find the final price of each product after applying the category discount. Return the result table ordered by `product_id` in ascending order.

## Example

### Input

Products table:

| product_id | category    | price |
|------------|-------------|-------|
| 1          | Electronics | 1000  |
| 2          | Clothing    | 50    |
| 3          | Electronics | 1200  |
| 4          | Home        | 500   |

Discounts table:

| category    | discount |
|-------------|----------|
| Electronics | 10       |
| Clothing    | 20       |

### Output

| product_id | final_price | category    |
|------------|-------------|-------------|
| 1          | 900         | Electronics |
| 2          | 40          | Clothing    |
| 3          | 1080        | Electronics |
| 4          | 500         | Home        |

### Explanation

- For product 1, it belongs to the Electronics category which has a 10% discount, so the final price is 1000 - (10% of 1000) = 900.
- For product 2, it belongs to the Clothing category which has a 20% discount, so the final price is 50 - (20% of 50) = 40.
- For product 3, it belongs to the Electronics category and receives a 10% discount, so the final price is 1200 - (10% of 1200) = 1080.
- For product 4, no discount is available for the Home category, so the final price remains 500.
- Result table is ordered by product_id in ascending order.

## Solution

```sql
SELECT
    p.product_id,
    COALESCE((p.price - (p.price * (d.discount / 100))), p.price) AS final_price,
    p.category
FROM Products p
LEFT JOIN Discounts d
    ON p.category = d.category
ORDER BY p.product_id ASC
```

This query joins the Products and Discounts tables, calculates the discounted price where applicable, and returns the results ordered by product_id.
