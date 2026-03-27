## 1. Basic Syntax and Usage

The ORDER BY clause is used to sort the result set in either ascending (ASC) or descending (DESC) order.

- Default Behavior: If no direction is specified, MySQL sorts in ascending order.

- Syntax:

```sql
SELECT column1, column2, ...
FROM table_name
ORDER BY column_name [ASC|DESC];
```

```sql
CREATE DATABASE db12;
USE db12;
-- Create a products table with various data types
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    category VARCHAR(50),
    price DECIMAL(10,2),
    stock_quantity INT,
    last_updated TIMESTAMP
);

-- Insert initial sample data
INSERT INTO products VALUES
(1, 'Laptop Pro', 'Electronics', 1299.99, 50, '2024-01-15 10:00:00'),
(2, 'Desk Chair', 'Furniture', 199.99, 30, '2024-01-16 11:30:00'),
(3, 'Coffee Maker', 'Appliances', 79.99, 100, '2024-01-14 09:15:00'),
(4, 'Gaming Mouse', 'Electronics', 59.99, 200, '2024-01-17 14:20:00'),
(5, 'Bookshelf', 'Furniture', 149.99, 25, '2024-01-13 16:45:00');

-- Display all records (unsorted)
SELECT * FROM products;

-- Sort by price in ascending order (ASC is optional as it's the default)
SELECT * FROM products ORDER BY price ASC;

-- Sort by last updated timestamp
SELECT * FROM products ORDER BY last_updated;
```

## 2. Sorting by Multiple Columns

You can sort by multiple columns by separating them with commas. This is useful when the first column has duplicate values.

```sql
-- Multiple column sorting (sort by category descending, then price descending)
SELECT * FROM products ORDER BY category DESC, price DESC;
```

## 3. Custom Sorting Techniques

A. Using the FIELD() Function
When you need a specific, non-alphabetical order(e.g., prioritizing "Electronics" over "Appliances"), use the FIELD() function.

```sql
-- Default category sorting
SELECT * FROM products ORDER BY category;

-- Custom category order using FIELD function
SELECT * FROM products
ORDER BY FIELD(category, 'Electronics','Appliances','Furniture'), price DESC;
```

B. Using CASE Statements
For complex logic, such as prioritizing "Low Stock" items with "High Prices" for a clearance sale, CASE allows you to assign numeric priorities to different conditions.

```sql
-- Simple conditional sorting for low stock and high price items
SELECT *,
    stock_quantity <= 50 AND price >= 200 AS priority_flag
FROM products
ORDER BY (stock_quantity <= 50 AND price >= 200) DESC;

-- Advanced priority-based sorting using CASE
SELECT *,
    CASE
        WHEN stock_quantity <= 50 AND price >= 200 THEN 1
        WHEN stock_quantity <= 50 THEN 2
        ELSE 3
    END AS priority
FROM products
ORDER BY priority;
```

## 4. Advanced Sorting Features

- Sorting by Column Position: You can use numbers (e.g.m ORDER BY 2) to refer the second column in the SELECT list. However, this is not recommended as it makes queries fragile if the table schema changes.

```sql
-- Sort using column position (4 represents the price column)
SELECT * FROM products ORDER BY 4;

-- Combining WHERE clause with ORDER BY
SELECT * FROM products
WHERE category = 'Electronics'
ORDER BY price;

-- Case-sensitive sorting using BINARY
SELECT * FROM products ORDER BY BINARY category;
```

- Sorting by Functions: You can sort based on calculated values, such as the LENGTH() of a string or specific date parts using YEAR().

```sql
-- Sort by product name length
SELECT * FROM products ORDER BY LENGTH(product_name);

-- Sort by day of the month from timestamp
SELECT * FROM products ORDER BY DAY(last_updated);

-- Using LIMIT with ORDER BY to find highest stock quantity
SELECT * FROM products
ORDER BY stock_quantity DESC
LIMIT 1;
```

- Null Handling: By default, NULL values appear first in ascending order. You can use IS NULL logic to push them to the end.

```sql
-- Add records with NULL values for demonstration
INSERT INTO products VALUES
(6, 'Desk Lamp', 'Furniture', NULL, 45, '2024-01-18 13:25:00'),
(7, 'Keyboard', 'Electronics', 89.99, NULL, '2024-01-19 15:10:00');

-- Basic NULL handling in ORDER BY
SELECT * FROM products ORDER BY price;

-- Explicit NULL handling
SELECT *,
    price IS NULL
FROM products
ORDER BY price IS NULL;

-- Sort by total value (price * quantity)
SELECT *,
    price * stock_quantity AS total_value
FROM products
ORDER BY total_value DESC;
```

## 5. Performance & Best Practices

- Indexing: To speed up sorting, ensure the columns used in ORDER BY are indexed.

- Explain Command: Use EXPLAIN before your query to see if MySQL is using an index or performing a "File Sort" (which is slower).

```sql
-- Examine query execution plan for multi-column sort
EXPLAIN SELECT * FROM products
ORDER BY category, price;

-- Compare with primary key sort performance
EXPLAIN SELECT * FROM products
ORDER BY product_id;
```

- Predictability: Never assume a default order without ORDER BY. Without it, the sequence depends on the storage engine and caching, which can lead to inconsistent results.