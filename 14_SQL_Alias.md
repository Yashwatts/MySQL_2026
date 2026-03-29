## 1. What is a MySQL Alias?

Aliases in MySQL are temporary names assigned to database tables, columns, or expressions to make them more readable and manageable. It does not change the actual name in the database.

```sql
-- Create and use the database
CREATE DATABASE db14;
USE db14;

-- Create employees table
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    salary DECIMAL(10,2),
    hire_date DATE
);

-- Insert initial employee data
INSERT INTO employees VALUES
    (1, 'John', 'Doe', 60000.00, '2020-01-15'),
    (2, 'Jane', 'Smith', 65000.00, '2019-11-20'),
    (3, 'Mike', 'Johnson', 55000.00, '2021-03-10');

-- View all employees
SELECT * FROM employees;
```

## 2. Column Aliases

Column aliases are used to give a column in your result set a different heading. This is particularly useful when:

- The original column name is technical or unclear (e.g., emp_id renamed to ID).
- You want to make the output more user-friendly.

**Syntax Example:**

```sql
SELECT emp_id AS ID FROM employees;
```

## 3. Aliases for Expressions and Functions

When you perform calculations or use functions, the default column name in the output is often the expression itself. Aliases help clean this up.

- **Calculations:** You can alias a math operation, such as calculating a 10% salary increment as NEW_SALARY.

```sql
SELECT salary, salary * 1.1 as new_salary from employees;
```

- **Functions:** When using functions like CONCAT to merge a first and last name, you can alias the result as Full_Name for better readability.

```sql
SELECT CONCAT(first_name, " ", last_name) AS full_name FROM employees;
```

```sql
-- Create departments table
CREATE TABLE departments (
    dept_id INT PRIMARY KEY,
    dept_name VARCHAR(50),
    location VARCHAR(50)
);

-- Insert department data
INSERT INTO departments VALUES
    (1, 'Engineering', 'New York'),
    (2, 'Marketing', 'Los Angeles'),
    (3, 'Finance', 'Chicago');

-- Add department reference to employees table
ALTER TABLE employees ADD COLUMN department_id INT;
```

## 4. Table Aliases

Table aliases are used to shorten table names, making the query easier to write and read, especially during Joins.

- They allow you to refer to a table by a single letter (e.g., employees as e).
- Joins: Aliases are essential when joining multiple tables to specify which table a column belongs to (e.g., e.first_name and d.dept_name).

```sql
-- Using table aliases in JOIN operations
SELECT 
    e.first_name,
    e.last_name,
    d.dept_name 
FROM employees AS e 
JOIN departments AS d 
    ON e.department_id = d.dept_id;
```

## 5. Key Technical Tips

- **The AS Keyword:** Using AS is common practice, but it is optional. You can create an alias by simply putting the temporary name after the original name.

- **Subqueries:** You can assign an alias to the result of a subquery (often called a "Derived Table") to reference its data in the main query.

Example:

```sql
SELECT avg_salary.average_salary
FROM (
    SELECT AVG(salary) AS average_salary 
    FROM employees
) AS avg_salary;
```

- **Case Statements:** Aliases can also be applied to CASE logic to name the resulting conditional column.

Example:

```sql
SELECT first_name,
        CASE
           WHEN age < 18 THEN 'Minor'
           ELSE 'Adult'
        END AS Age_Category
FROM students;
```

---

**Summary:** The primary purpose of an alias is to simplify queries and improve output readability for the user.