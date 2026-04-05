## 1. Basic Syntax of UPDATE

- The UPDATE statement is used to modify existing records in a table.
- Standard Syntax:

```sql
UPDATE table_name 
SET column1 = value1, column2 = value2, ...
WHERE condition;
```

- SET: Specifies the columns and values to update.
- WHERE: Specifies which records should be updated. If omitted, all records in the table will be updated.

## 2. Safe Update Mode

- By default, many SQL editors (like MySQL Workbench) have Safe Update Mode enabled.
- Why? It prevents you from accidentally updating or deleting all records in a table.
- Constraint: You cannot run an UPDATE without a WHERE clause that uses a Key Column (like a Primary Key).
- Workaround: To update all rows while Safe Mode is on, you can use a dummy condition like WHERE id > 0.

## 3. Updating Multiple Columns

- You can update multiple fields in a single row by separating them with commas.
- Example: Updating both price and stock_quantity for a specific product ID.

## 4. Tracking Changes (Auto-Timestamp)

- To track when a row was last modified, you can use the TIMESTAMP data type with a special attribute:
- The Command: ON UPDATE CURRENT_TIMESTAMP
- Functionality: Whenever any column in that row is changed, MySQL automatically updates the timestamp column to the current time.
- Altering Existing Table: Use ALTER TABLE and MODIFY to add this behavior to an existing column.

## 5. Advanced Update Controls

- LIMIT Clause
- You can restrict the number of rows updated using LIMIT.
    - Example: UPDATE ... LIMIT 2; only updates the first two rows it finds.

- ORDER BY with UPDATE
- To ensure the "first two rows" are specific ones (e.g., the cheapest products), combine UPDATE with ORDER BY.

## 6. Constraints and Errors

- Primary Key Violations: You cannot update a column to a value that already exists if that column is a Primary Key. This will result in a "Duplicate entry" error.
- Data Truncation: If you update a decimal value that exceeds the defined precision (e.g., more than 2 decimal places), MySQL may round the value and issue a warning.

```sql
-- Create the database
CREATE DATABASE store_inventory;

-- Switch to the new database
USE store_inventory;

-- Create products table
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(50) NOT NULL,
    category VARCHAR(20),
    price DECIMAL(10, 2),
    stock_quantity INT,
    last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Insert initial data
INSERT INTO products (product_id, product_name, category, price, stock_quantity)
VALUES 
    (1, 'Laptop', 'Electronics', 899.99, 25),
    (2, 'Desk Chair', 'Furniture', 149.50, 40),
    (3, 'Coffee Maker', 'Appliances', 79.99, 15),
    (4, 'Headphones', 'Electronics', 129.99, 30),
    (5, 'Desk Lamp', 'Furniture', 24.99, 50);

-- View all products
SELECT * FROM products;

-- Apply 10% discount to all products
UPDATE products SET price = price * 0.9 WHERE product_id > 0;

-- Update specific product prices
UPDATE products
SET price = 999.99
WHERE product_id = 1;

UPDATE products
SET price = 89.99, stock_quantity = 20
WHERE product_id = 3;

-- Modify the last_updated column to automatically update on changes
ALTER TABLE products
MODIFY last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP;

-- View updated products
SELECT * FROM products;

-- Update laptop price and quantity
UPDATE products
SET price = 199, stock_quantity = 1
WHERE product_id = 1;

-- View products after update
SELECT * FROM products;

-- Apply 90% discount to first two products
UPDATE products SET price = price * 0.1 WHERE product_id > 0 LIMIT 2;

-- View products after discount
SELECT * FROM products;

-- Attempt to update product_id (will cause a primary key constraint violation)
UPDATE products
SET product_id = 1
WHERE product_id = 2;

-- View final state of products table
SELECT * FROM products;
```