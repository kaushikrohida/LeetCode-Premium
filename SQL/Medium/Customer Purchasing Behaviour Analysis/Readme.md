# Customer Purchasing Behavior Analysis

## Problem Description

Analyze customer purchasing behavior by calculating various metrics for each customer.

## Tables

### Transactions

| Column Name      | Type    |
|------------------|---------|
| transaction_id   | int     |
| customer_id      | int     |
| product_id       | int     |
| transaction_date | date    |
| amount           | decimal |

`transaction_id` is the unique identifier for this table.
Each row contains information about a transaction, including the customer ID, product ID, date, and amount spent.

### Products

| Column Name | Type    |
|-------------|---------|
| product_id  | int     |
| category    | varchar |
| price       | decimal |

`product_id` is the unique identifier for this table.
Each row contains information about a product, including its category and price.

## Task

For each customer, calculate:

1. The total amount spent
2. The number of transactions
3. The number of unique product categories purchased
4. The average amount spent
5. The most frequently purchased product category (if there's a tie, choose the one with the most recent transaction)
6. A loyalty score defined as: (Number of transactions * 10) + (Total amount spent / 100)

Round `total_amount`, `avg_transaction_amount`, and `loyalty_score` to 2 decimal places.

Return the result table ordered by `loyalty_score` in descending order, then by `customer_id` in ascending order.

## Example

### Input

Transactions table:

| transaction_id | customer_id | product_id | transaction_date | amount |
|----------------|-------------|------------|------------------|--------|
| 1              | 101         | 1          | 2023-01-01       | 100.00 |
| 2              | 101         | 2          | 2023-01-15       | 150.00 |
| 3              | 102         | 1          | 2023-01-01       | 100.00 |
| 4              | 102         | 3          | 2023-01-22       | 200.00 |
| 5              | 101         | 3          | 2023-02-10       | 200.00 |

Products table:

| product_id | category | price  |
|------------|----------|--------|
| 1          | A        | 100.00 |
| 2          | B        | 150.00 |
| 3          | C        | 200.00 |

### Output

| customer_id | total_amount | transaction_count | unique_categories | avg_transaction_amount | top_category | loyalty_score |
|-------------|--------------|-------------------|-------------------|------------------------|--------------|---------------|
| 101         | 450.00       | 3                 | 3                 | 150.00                 | C            | 34.50         |
| 102         | 300.00       | 2                 | 2                 | 150.00                 | C            | 23.00         |

## Solution

```sql
with cte as (
select
    t.customer_id,
    sum(t.amount) as total_amount,
    count(t.transaction_id) as transaction_count,
    count(distinct p.category) as unique_categories,
    avg(t.amount) as avg_transaction_amount,
    round(((count(t.transaction_id) * 10) + (sum(t.amount) / 100)), 2) as loyalty_score
from Transactions t
Join Products p
    on t.product_id = p.product_id
group by 1
),

cte2 as(
select
    t.customer_id,
    p.category,
    dense_rank () over (partition by customer_id order by count(*) desc, max(t.transaction_date) desc) dr
from Transactions t
Join Products p
    on t.product_id = p.product_id
group by 1, 2
)

select
    c1.customer_id,
    round(c1.total_amount, 2) as total_amount,
    c1.transaction_count,
    c1.unique_categories,
    round(c1.avg_transaction_amount, 2) as avg_transaction_amount,
    c2.category as top_category,
    round(c1.loyalty_score, 2) as loyalty_score
from cte c1
Join cte2 c2
    on c1.customer_id = c2.customer_id
where 
    c2.dr = 1
order by c1.loyalty_score desc, c1.customer_id asc
