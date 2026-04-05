## SQL REPLACE Statement Overview

- The REPLACE statement in MySQL is a powerful command that functions as a combination of INSERT and DELETE.
- It is primarily used to simplify the process of updating or inserting records based on unique values.


## 1. Core Logic: How it Works

- The behavior of the REPLACE statement depends on whether a record with the same Primary Key or Unique Index already exists in the table:

- If a duplicate exists: The statement first deletes the existing row and then inserts the new row with the updated values.

- If no duplicate exists: It simply acts as a standard INSERT statement and adds the new row to the table.


## 2. Syntax

- The syntax for REPLACE is identical to the INSERT statement; you simply substitute the word INSERT with REPLACE.

Basic Example:
```sql
REPLACE INTO products (product_id, product_name, category, price)
VALUES (5, 'Mic', 'Electronics', 500);
```


## 3. Key Use Cases

- Handling Errors: Unlike INSERT, which throws an error if you try to add a duplicate primary key, REPLACE handles the conflict by overwriting the old data.

- Replacing via SELECT: You can use the result of a SELECT statement to replace data in another table. This is useful for syncing data between two tables where you want new records added and existing ones updated.


## 4. Practical Demonstration Summary

- Conflict Scenario: When we try to insert a new product (Mic) with an existing ID (5), a standard INSERT failed. Using REPLACE successfully removed the old entry (Desk Chair) and inserted the Mic.

- Bulk Replacement: Replacing the entire content of a products table with data from a products_2 table using a subquery, ensuring all matching IDs are updated and new IDs are added.


```sql
-- Create and use database
CREATE DATABASE replace_demo;
USE replace_demo;

-- Create products table
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    category VARCHAR(50),
    price DECIMAL(10, 2),
    stock_quantity INT,
    last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- Insert initial product data
INSERT INTO products (product_id, product_name, category, price, stock_quantity)
VALUES 
    (1, 'Laptop', 'Electronics', 899.99, 25),
    (2, 'Smartphone', 'Electronics', 599.99, 50),
    (3, 'Coffee Maker', 'Kitchen', 79.99, 30),
    (4, 'Running Shoes', 'Sportswear', 129.99, 40),
    (5, 'Desk Chair', 'Furniture', 189.99, 15);

-- Replace an existing product (ID 5)
-- This will DELETE the existing record and INSERT a new one
REPLACE INTO products (product_id, product_name, category, price, stock_quantity)
VALUES
    (5, 'Mic', 'Electronics', 500, 12);
    
-- Add a new product (ID 6) with REPLACE
-- Since the ID doesn't exist, this works like a regular INSERT
REPLACE INTO products (product_id, product_name, category, price)
VALUES
    (6, 'Camera', 'Electronics', 5000);
    
-- View the updated products table
SELECT * FROM products;

-- Create a second products table with an additional supplier column
CREATE TABLE products2 (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    category VARCHAR(50),
    price DECIMAL(10, 2),
    stock_quantity INT,
    supplier VARCHAR(100) -- Extra column
);
    
-- Insert data into products2 including both existing and new product IDs
INSERT INTO products2 (product_id, product_name, category, price, stock_quantity, supplier)
VALUES 
    (2, 'Ultra Smartphone', 'Electronics', 899.99, 40, 'TechCorp'), -- Existing ID (2)
    (4, 'Pro Running Shoes', 'Sportswear', 149.99, 35, 'SportMaster'), -- Existing ID (4)
    (7, 'Bluetooth Speaker', 'Electronics', 79.99, 60, 'SoundWave'), -- New ID
    (8, 'Gaming Mouse', 'Computer Accessories', 49.99, 100, 'GamerZone'), -- New ID
    (9, 'Portable Monitor', 'Electronics', 199.99, 25, 'DisplayTech'); -- New ID
    
-- View both product tables before the bulk REPLACE operation
SELECT * FROM products;
SELECT * FROM products2;

-- Use REPLACE with SELECT to perform bulk replace operation
-- This will update products with IDs 2 and 4, and insert products with IDs 7, 8, and 9
REPLACE INTO products (product_id, product_name, category, price, stock_quantity)
SELECT product_id, product_name, category, price, stock_quantity
FROM products2;
    
-- View the final products table after bulk REPLACE operation
SELECT * FROM products;
```