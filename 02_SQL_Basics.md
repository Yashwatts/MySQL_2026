# What is Structured Query Language (SQL)?

SQL is a programming language that is used to manage and manipulate relational databases.
SQL deals with structured data, that is data which is organized in tables following a strict format.

## Why We Use SQL?

- Create Tables
- Modify Tables
- Retrieve Data
- Delete Data
- Modify Access

## Main Purpose of SQL

Fast Retrieval of Data

## Main Categories of SQL Commands

- DDL (Data Definition Language)
- DML (Data Manipulation Language)
- DCL (Data Control Language)
- TCL (Transaction Control Language)

## Data Definition

SQL provides commands to define the structure of a database, collectively known as Data Definition Language (DDL). Some common DDL commands include:

- CREATE: Used to create database objects like tables, views, indexes, etc.
- ALTER: Used to modify existing database objects (e.g., add or remove a column from a table.)
- DROP: Used to delete entire database objects like tables, views, or indexes.
- TRUNCATE: Removes all rows from a table but does not delete the table itself.
- RENAME: Changes the name of a database object.

## Manipulating Data

Data Manipulation Language (DML) commands let you insert, update or delete data within the database tables. These commands include:

- SELECT: Retrieves data from one or more tables.
- INSERT: Adds new records into a table.
- UPDATE: Modifies existing records in a table.
- DELETE: Removes records from a table.

## Data Control Language (DCL)

DCL commands are used to control access to data in a database.

- GRANT: Gives a user access privileges to database objects.
- REVOKE: Removes access privileges from a user.

## Transaction Control Language (TCL)

TCL commands are used to manage transactions within a database, ensuring data integrity and consistency.

- COMMIT: Saves all changes made during the current transaction.
- ROLLBACK: Undoes changes made during the current transaction.
- SAVEPOINT: Sets a point within a transaction to which you can roll back.
- SET TRANSACTION: Configures transaction properties.

## Key Features and Advantages of Using SQL

1. Simplicity: SQL uses an English-like syntax (e.g., SELECT, UPDATE, WHERE), making it relatively easy to learn.
2. Ubiquity: Nearly every relational database system supports SQL. Once you learn core SQL, you can apply it to MySQL, PostgreSQL, Oracle and more.
3. Flexibility: You can handle complex queries involving multiple joined tables, subqueries, window functions and more.
4. Performance: When used properly with indexes and query optimization, SQL can query large volumes of data very efficiently.
5. Data Integrity: Features like transactions, constraints (PRIMARY KEY, FOREIGN KEY, UNIQUE) and ACID compliance ensure that your data remains accurate and consistent.

## SQL in MySQL

MySQL is one of the most popular open-source relational database systems that implements SQL. Here's how SQL typically is used with MySQL:

1. Write SQL statements in a client (MySQL CLI or a GUI tool like MySQL Workbench).
2. Execute the statements. MySQL's SQL engine interprets, optimize and runs them against the database.
3. Receive results, either in text form (CLI) or tabular data (GUI).

## Example

A simple MySQL command-line session might look like this:

```sql
-- Use the desired database
USE my_database;

-- Create a table
CREATE TABLE products {
    product_id INT AUTO_INCREMENT PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    price DECIMAL(10, 2) NOT NULL
};

-- Insert records
INSERT INTO products (product_name, price)
VALUES ('Laptop', 999.99),
       ('Headphones', 49.99),
       ('Monitor', 199.99);

-- Select records
SELECT product_id, product_name, price
FROM products
ORDER BY price DESC;
```