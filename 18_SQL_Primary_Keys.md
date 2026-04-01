# Introduction to Primary Keys

- A Primary Key is a column (or a set of columns) that uniquely identifies each row in a database table.

## Core Characteristics

- Uniqueness: No two rows can have the same value in the primary key column.
- Not Null: The primary key field cannot be empty; it must contain a value.
- Stability: The values should be stable and not change frequently.

## Implementation & Syntax

- Primary keys act as reference points for connecting multiple tables and enable faster data retrieval.

### 1. Inline Definition

- You can define a primary key directly when creating a column:

```sql
CREATE TABLE Students (
    StudentID INT PRIMARY KEY,
    StudentName VARCHAR(50)
);
```

- Creating a table with a simple primary key

```sql
CREATE TABLE students (
    student_id INT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100)
);
```

- Inserting records with valid primary keys

```sql
INSERT INTO students (student_id, first_name, last_name, email)
VALUES 
(1, 'John', 'Smith', 'john.smith@example.com'),
(2, 'Maria', 'Garcia', 'maria.garcia@example.com'),
(3, 'Ahmed', 'Khan', 'ahmed.khan@example.com');
```

- Demonstrating primary key constraint - This will fail

```sql
INSERT INTO students (student_id, first_name, last_name, email)
VALUES (1, 'Jane', 'Doe', 'jane.doe@example.com');
-- Error Code: 1062. Duplicate entry '1' for key 'PRIMARY'
```

### 2. Auto-Increment

- To avoid manually managing unique IDs, use AUTO_INCREMENT. The database will automatically generate a new unique number for every entry.

```sql
CREATE TABLE Products (
    ProductID INT PRIMARY KEY AUTO_INCREMENT,
    ProductName VARCHAR(100)
);
```

- Creating a table with an auto-increment primary key

```sql
CREATE TABLE products (
    product_id INT AUTO_INCREMENT PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    price DECIMAL(10, 2) NOT NULL,
    description TEXT
);
```

- With auto-increment, we don't need to specify the primary key value

```sql
INSERT INTO products (product_name, price, description)
VALUES 
('Laptop', 1299.99, 'High-performance laptop'),
('Smartphone', 799.99, 'Latest model smartphone'),
('Headphones', 199.99, 'Noise-cancelling headphones');
```

- View the auto-generated IDs

```sql
SELECT * FROM products;
```

### 3. Adding to an Existing Table

- If a table was created without a primary key, use the ALTER TABLE command to add one later.

```sql
ALTER TABLE Suppliers ADD PRIMARY KEY (SupplierID);
```

- Creating a table with a primary key defined separately

```sql
CREATE TABLE orders (
    order_id INT,
    customer_id INT,
    order_date DATE NOT NULL,
    total_amount DECIMAL(10, 2) NOT NULL,
    PRIMARY KEY (order_id)
);
```

- Create table without primary key

```sql
CREATE TABLE suppliers (
    supplier_id INT,
    supplier_name VARCHAR(100) NOT NULL,
    contact_person VARCHAR(100)
);
```

- Adding a primary key to an existing table

```sql
ALTER TABLE suppliers
ADD PRIMARY KEY (supplier_id);
```

## Composite Primary Keys

- When a single column isn't enough to uniquely identify a row, you can combine two or more columns. This is known as a Composite Primary Key.
- Example: In an enrollment table, the combination of StudentID and CourseID ensures a student cannot enroll in the same course twice.
- Syntax:

```sql
CREATE TABLE Enrollments (
    StudentID INT,
    CourseID INT,
    PRIMARY KEY (StudentID, CourseID)
);
```

- Creating a table with a composite primary key (multiple columns)

```sql
CREATE TABLE enrollments (
    student_id INT,
    course_id INT,
    enrollment_date DATE NOT NULL,
    grade VARCHAR(2),
    PRIMARY KEY (student_id, course_id)
);
```

- Insert records with unique combinations of the composite key

```sql
INSERT INTO enrollments (student_id, course_id, enrollment_date, grade)
VALUES 
(1, 101, '2023-01-15', 'A'),
(1, 102, '2023-01-15', 'B+'),  -- Same student, different course - OK
(2, 101, '2023-01-16', 'A-'),  -- Different student, same course - OK
(3, 103, '2023-01-17', 'B');
```

- This will fail - duplicate composite key (student_id + course_id)

```sql
INSERT INTO enrollments (student_id, course_id, enrollment_date, grade)
VALUES (1, 101, '2023-02-01', 'C');
-- Error: Duplicate entry '1-101' for key 'PRIMARY'
```

## Internal Working: Clustered Index

- When you define a primary key in MySQL (using the InnoDB engine), it creates a Clustered Index.
- Clustered vs Non-Clustered:
    - Clustered Index: Like a dictionary, where the actual data is physically stored in the order of the keys.
    - Non-Clustered Index: Like a book's index at the back; it contains pointers to the data stored in a different location.
- Data Structure: MySQL uses a B+ Tree structure to store this data. The actual row data is stored in the "Leaf Nodes" while "Root" and "Branch" nodes contain keys and pointers for fast searching.

## Storage Architecture: Pages

- Data in MySQL isn't stored row-by-row on the disk; it is organized into containers called Pages.
- Fixed Size: The default size of a page is 16 KB.
- Contents: A page acts as a container for the B+ Tree nodes (Root, Branch, or Leaf nodes).
- Splitting: If a page becomes full due to too much data, MySQL automatically "splits" it and creates a new page to maintain the structure.

## Memory Management: The Buffer Pool

- To avoid hitting the slow physical disk (Hard Drive/SSD) for every operation, MySQL maintains a memory area called the Buffer Pool.
- The Cache Mechanism: The Buffer Pool acts as a cache. When you request data, MySQL first checks if the required Page is already in the Buffer Pool.
- Efficiency: Frequently accessed pages (frequently used data) are kept in this memory. This allows for near-instant retrieval compared to reading from the disk.
- Internal Handling: This entire process is managed by the InnoDB storage engine (the default engine for MySQL).

## Summary of the Data Flow

1. Query is sent to the database.
2. Database checks the Buffer Pool (Memory) for the relevant Page.
3. If found (Cache Hit), data is returned immediately.
4. If not found (Cache Miss), the database fetches the 16 KB Page from the disk, puts it in the Buffer Pool for future use, and then returns the data.

## Best Practices

- Always include a Primary Key: Every table should have one for integrity and speed.
- Keep it Simple: Use integers (INT or BIGINT) rather than long strings for better performance.
- Efficiency: Operations like UPDATE or DELETE are significantly faster when targeting a primary key.