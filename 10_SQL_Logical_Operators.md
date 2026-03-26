## 1. Overview of Logical Operators

Logical operators are used to combine multiple conditions in a WHERE clause to filter specific rows from a table.

- AND: Used when all combined conditions must be true for a row to be included.

- OR: Used when at least one of the conditions must be true.

- NOT: Used to negate a condition (returns rows where the condition is false).

```sql
CREATE DATABASE company_db;
USE company_db;
CREATE TABLE employees (
    emp_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50),
    age INT,
    department VARCHAR(50),
    salary DECIMAL(10, 2),
    city VARCHAR(50)
);
INSERT INTO employees (name, age, department, salary, city) VALUES
('Alice Johnson', 30, 'HR', 50000, 'New York'),
('Bob Smith', 25, 'IT', 70000, 'Los Angeles'),
('Charlie Brown', 35, 'IT', 80000, 'New York'),
('David Wilson', 40, 'Finance', 90000, 'Chicago'),
('Emily Davis', 28, 'HR', 48000, 'San Francisco'),
('Franklin Moore', 32, 'IT', 75000, 'Los Angeles'),
('Grace Adams', 45, 'Finance', 95000, 'Chicago');

SELECT * FROM employees;
```

Examples:

```sql
-- Find employees who work in the IT department and earn more than $70,000
SELECT * FROM employees
WHERE department = 'IT' AND salary > 70000;

-- Find employees who work in the HR department OR live in New York
SELECT * FROM employees
WHERE department = 'HR' OR city = 'New York';

-- Find employees who are not in the Finance department
SELECT * FROM employees
WHERE NOT department = 'Finance';

-- Find employees who are in IT and earn more than $70,000 OR work in Finance
SELECT * FROM employees
WHERE (departmet = 'IT' AND salary > 70000) OR department = 'Finance';

-- Find employees who are not in the IT department and do not live in Chicago
SELECT * FROM employees
WHERE NOT department = 'IT' AND NOT city = 'Chicago';
```