# Core Concept: What is a Foreign Key?

- A Foreign Key is a column (or group of columns) in one table that refers to the Primary Key in another table. It acts as a link between the two, establishing a formal relationship.
- Parent Table: The table containing the Primary Key being referenced.
- Child Table: The table containing the Foreign Key.

## Why use Foreign Keys?

1. Referential Integrity: Ensures relationships remain consistent. For example, you cannot add an order for a customer who doesn't exist.
2. Data Validation: Prevents invalid data entry into the child table if the corresponding value isn't in the parent table.
3. Delete Prevention: Prevents deleting a record from the parent table if it is still being referenced by records in the child table.

## Types of Database Relationships

- There are 3 primary ways table can relate to each other:

| Relationship | Description | Example |
|---|---|---|
| One-to-One | One record in Table A relates to exactly one in Table B. | Employee ↔️ Employee Details |
| One-to-Many | One record in Table A relates to multiple in Table B. | Department ↔️ Many Employees |
| Many-to-Many | Multiple records in Table A relate to multiple in Table B. | Students ↔️ Courses (uses a Junction Table) |

- 1. One-to-One (1:1): Each record in Table A relates to exactly one record in Table B

```sql
CREATE TABLE employee_details (
    employee_id INT NOT NULL,
    passport_number VARCHAR(20),
    marital_status VARCHAR(20),
    emergency_contact VARCHAR(100),
    PRIMARY KEY (employee_id),
    FOREIGN KEY (employee_id) REFERENCES employees(employee_id)
);
```

- 2. One-to-Many (1:N): Each record in Table A relates to multiple records in Table B

```sql
CREATE TABLE employees (
    employee_id INT NOT NULL,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    department_id INT,
    PRIMARY KEY (employee_id),
    FOREIGN KEY (department_id) REFERENCES departments(department_id)
);
```

- Here, multiple employee records can reference the same department_id

- 3. Many-to-Many (N:M): Multiple records in Table A relate to multiple records in Table B

```sql
-- Create Students table
CREATE TABLE Students (
    student_id INT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL
);

-- Create Courses table
CREATE TABLE Courses (
    course_id INT PRIMARY KEY,
    course_name VARCHAR(100) NOT NULL,
    instructor VARCHAR(100) NOT NULL
);

-- Create Enrollments junction table with foreign keys
CREATE TABLE Enrollments (
    enroll_id INT PRIMARY KEY,
    student_id INT NOT NULL,
    course_id INT NOT NULL,
    grade VARCHAR(5),
    FOREIGN KEY (student_id) REFERENCES Students(student_id),
    FOREIGN KEY (course_id) REFERENCES Courses(course_id)
);
```

## SQL Syntax and Commands

### 1. Creating a Foreign Key (Table Creation)

- To define a foreign key while creating a table, use the FOREIGN KEY and REFERENCES keywords:

```sql
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(50),
    dept_id INT,
    FOREIGN KEY (dept_id) REFERENCES departments(dept_id)
);
```

### 2. Adding a Foreign Key (Existing Table)

- If the table already exists, use the ALTER TABLE command:

```sql
ALTER TABLE projects 
ADD FOREIGN KEY (manager_id) REFERENCES employees(emp_id);
```

### 3. Dropping a Foreign Key

- To remove a constraint, you must use the specific constraint name:

```sql
ALTER TABLE projects 
DROP FOREIGN KEY constraint_name;
```

- Pro Tip: If you don't know the constraint name, use SHOW CREATE TABLE table_name; to find the exact metadata and naming used by the database.

## PRACTICAL IMPLEMENTATION

```sql
-- Create a database
CREATE DATABASE company_db;
USE company_db;

-- Create the parent table (departments)
CREATE TABLE departments (
    department_id INT NOT NULL,
    department_name VARCHAR(100) NOT NULL,
    location VARCHAR(100),
    PRIMARY KEY (department_id)
);

-- Create the child table with a foreign key (employees)
CREATE TABLE employees (
    employee_id INT NOT NULL,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100),
    hire_date DATE,
    salary DECIMAL(10,2),
    department_id INT,
    PRIMARY KEY (employee_id),
    FOREIGN KEY (department_id) REFERENCES departments(department_id)
);
```

## INSERT SAMPLE DATA

```sql
-- Insert department data
INSERT INTO departments (department_id, department_name, location)
VALUES 
    (1, 'Human Resources', 'Floor 1'),
    (2, 'Marketing', 'Floor 2'),
    (3, 'Engineering', 'Floor 3'),
    (4, 'Finance', 'Floor 1');

-- Insert employee data
INSERT INTO employees (employee_id, first_name, last_name, email, hire_date, salary, department_id)
VALUES
    (101, 'John', 'Smith', 'john.smith@company.com', '2018-06-20', 55000.00, 1),
    (102, 'Sarah', 'Johnson', 'sarah.johnson@company.com', '2019-03-15', 62000.00, 2),
    (103, 'Michael', 'Williams', 'michael.williams@company.com', '2020-01-10', 75000.00, 3),
    (104, 'Emily', 'Brown', 'emily.brown@company.com', '2019-11-05', 68000.00, 3),
    (105, 'David', 'Jones', 'david.jones@company.com', '2021-02-28', 58000.00, 4),
    (106, 'Jessica', 'Davis', 'jessica.davis@company.com', '2020-07-16', 61000.00, 2),
    (107, 'Robert', 'Miller', 'robert.miller@company.com', '2018-09-12', 72000.00, 3);

-- View employee data
SELECT * FROM employees;
```

## DEMONSTRATING FOREIGN KEY CONSTRAINT

```sql
-- Attempt to insert an employee with non-existent department_id (this will fail)
INSERT INTO employees (employee_id, first_name, last_name, email, hire_date, salary, department_id)
VALUES
    (145, 'John', 'Smith', 'john.smith@company.com', '2018-06-20', 55000.00, 69);
-- Error: Cannot add or update a child row: a foreign key constraint fails

-- Insert an employee with NULL department_id (allowed if the foreign key allows NULL)
INSERT INTO employees (employee_id, first_name, last_name, email, hire_date, salary, department_id)
VALUES 
    (108, 'Thomas', 'Wilson', 'thomas.wilson@company.com', '2022-04-10', 65000.00, NULL);
```

## ADDING AND REMOVING FOREIGN KEYS

```sql
-- Create a projects table
CREATE TABLE projects (
    project_id INT NOT NULL,
    project_name VARCHAR(100) NOT NULL,
    start_date DATE,
    end_date DATE,
    manager_id INT,
    PRIMARY KEY (project_id)
);

-- Add a foreign key constraint after table creation
ALTER TABLE projects
ADD FOREIGN KEY (manager_id) REFERENCES employees(employee_id);

-- View the table structure including the foreign key
SHOW CREATE TABLE projects;

-- Remove a foreign key constraint
ALTER TABLE projects DROP FOREIGN KEY projects_ibfk_1;

-- Verify the foreign key was removed
SHOW CREATE TABLE projects;
```

## Important Behavior Notes

- Handling Nulls: By default, a Foreign Key column can accept NULL values unless specifically marked as NOT NULL. This represents an "optional relationship"-for example, a new employee who hasn't been assigned a department yet.
- Naming Constraints: You can manually name your foreign key constraints using the CONSTRAINT keyword to make them easier to manage later

## Practice Challenge

- Task: Create a table for employee_skills that includes a foreign key referencing the employees table.
- Practice Solution: Since one employee can have multiple skills, this is a One-to-Many relationship.

```sql
CREATE TABLE employee_skills (
    skill_id INT AUTO_INCREMENT PRIMARY KEY,
    skill_name VARCHAR(50) NOT NULL,
    proficiency_level ENUM('Beginner', 'Intermediate', 'Expert'),
    emp_id INT, -- This is our Foreign Key column
    
    -- Defining the relationship
    CONSTRAINT fk_employee 
    FOREIGN KEY (emp_id) 
    REFERENCES employees(emp_id)
    ON DELETE CASCADE -- Optional: deletes skills if the employee is removed
);
```

## Key Components

- emp_id: This column must have the same data type as the emp_id in the employees table.
- CONSTRAINT fk_employee: Giving the foreign key a name makes it much easier to drop or alter later.
- REFERENCES: This tells SQL exactly which table and column to "look at" to validate the data.

## How Junction Tables Work (Many-to-Many)

- A Junction Table (also called a Bridge or Join Table) is used when you have a Many-to-Many relationship.
- The Problem: You can't easily link Students and Courses directly. A student has many courses, and a course has many students. Putting a foreign key in either table would create a mess of duplicate data.
- The Solution: Create a third table that sits in the middle.

### How it's structured:

1. It holds two Foreign Keys: One points to Table A (Students) and the other points to Table B (Courses).
2. Composite Primary Key: Often, the two foreign keys combined act as the Primary Key for the junction table, ensuring a student can't enroll in the same course twice.
3. Extra Data: You can store "relationship-specific" data here, such as the enrollment_date or the grade the student received in that specific course.

### Example:

- The enrollments table acts as the junction:
- student_id -> References students table.
- course_id -> References courses table.