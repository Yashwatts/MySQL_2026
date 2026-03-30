## 1. String Functions

- Used for manipulating and formatting text data.
    - Create and use database for string function examples:
        ```sql
        CREATE DATABASE StringFunctionsDB;
        USE StringFunctionsDB;
        ```
    - Create employees table for string function demonstrations:
        ```sql
        CREATE TABLE employees (
            emp_id INT AUTO_INCREMENT PRIMARY KEY,
            first_name VARCHAR(50),
            last_name VARCHAR(50),
            email VARCHAR(100),
            department VARCHAR(50)
        );
        ```
    - Insert sample employee data:
        ```sql
        INSERT INTO employees (first_name, last_name, email, department) VALUES
        ('John', 'Doe', 'john.doe@example.com', 'Marketing'),
        ('Jane', 'Smith', 'jane.smith@example.com', 'Sales'),
        ('Michael', 'Johnson', 'michael.johnson@example.com', 'IT'),
        ('Emily', 'Davis', 'emily.davis@example.com', 'HR'),
        ('Chris', 'Brown', 'chris.brown@example.com', 'Finance');
        ```
    - `CONCAT(str1, str2, ...)`: Joins multiple strings into one.
        - Example: Combine first and last names into full name
            ```sql
            SELECT CONCAT(first_name, " ", last_name) AS full_name FROM employees;
            ```
    - `LENGTH()` vs `CHAR_LENGTH()`: `LENGTH()` returns the size in bytes, while `CHAR_LENGTH()` returns the number of characters (important for multi-byte characters like Japanese).
        - Example: Get the length of the first name
            ```sql
            SELECT first_name, LENGTH(first_name) AS name_length FROM employees;
            ```
        - Demonstrate difference with ASCII and multibyte characters
            ```sql
            SELECT LENGTH('hello') AS length_in_bytes;  -- Returns 5 (bytes)
            SELECT LENGTH('こんにちは') AS multibyte_length;  -- Returns more than 5 because each character is multiple bytes
            SELECT CHAR_LENGTH('hello') AS char_count;  -- Returns 5 (characters)
            SELECT CHAR_LENGTH('こんにちは') AS multibyte_char_count;  -- Returns 5 (characters)
            ```
    - `UPPER()` & `LOWER()`: Converts string to uppercase or lowercase.
        - Example: Convert first names to uppercase and lowercase
            ```sql
            SELECT first_name, UPPER(first_name) AS uppercase, LOWER(first_name) AS lowercase FROM employees;
            ```
    - `TRIM()`: Removes leading and trailing spaces.
        - Example: Remove leading and trailing spaces
            ```sql
            SELECT TRIM(UPPER('      ok.    ')) AS trimmed_sample;
            ```
    - `SUBSTRING(str, start, len)`: Extracts a specific part of a string.
        - Example: Extract the first three characters of first names
            ```sql
            SELECT first_name, SUBSTRING(first_name, 1, 3) AS first_three_chars FROM employees;
            ```
    - `LOCATE()`: Find the position of character in a string
        - Example: Find the position of character 'a' in first names
            ```sql
            SELECT first_name, LOCATE('a', first_name) AS position_of_a FROM employees;
            SELECT first_name, LOCATE('ch', first_name) AS position_of_ch FROM employees;
            ```
    - `REPLACE(str, from_str, to_str)`: Replaces occurrences of a substring.
        - Example: Replace domain in email addresses
            ```sql
            SELECT first_name, REPLACE(email, 'example.com', 'gmail.com') AS new_email FROM employees;
            ```
    - `REVERSE()`: Reverses the characters in a string.
        - Example: Reverse the characters in first names
            ```sql
            SELECT first_name, REVERSE(first_name) AS reversed_name FROM employees;
            ```
    - `LEFT()` & `RIGHT()`: To extract letters from leftmost or rightmost
        - Example: Get the first two and last two characters of first names
            ```sql
            SELECT first_name, LEFT(first_name, 2) AS first_two, RIGHT(first_name, 2) AS last_two FROM employees;
            ```
    - `ASCII()`: Returns the ASCII value of the character. If a string is given then returns the ASCII of the first character of that string.
        - Example: Get ASCII value of the first character in first names (regular and lowercase)
            ```sql
            SELECT first_name, ASCII(first_name) AS ascii_value, ASCII(LOWER(first_name)) AS ascii_lowercase_value FROM employees;
            ```
    - `SOUNDEX()`: Returns a character code based on how a word sounds, useful for finding similar sounding names (e.g., "Smith" and "Smyth").
        - Example: Compare phonetically similar strings
            ```sql
            SELECT SOUNDEX('Smith') AS smith_soundex;  -- Returns 'S530'
            SELECT SOUNDEX('Smyth') AS smyth_soundex;  -- Also returns 'S530'
            SELECT SOUNDEX('Robert') AS robert_soundex;  -- Returns 'R163'
            SELECT SOUNDEX('Rupert') AS rupert_soundex;  -- Also returns 'R163'
            ```
        - Find employees with names that sound like "Jane"
            ```sql
            SELECT * FROM employees
            WHERE SOUNDEX('jane') = SOUNDEX(first_name);
            ```

## 2. Field Function

- `FIELD()` assigns a position number to a value based on a custom list, which can be used for custom sorting.
    - Create products database for FIELD function demonstration:
        ```sql
        CREATE DATABASE db12;
        USE db12;
        ```
    - Create products table:
        ```sql
        CREATE TABLE products (
            product_id INT PRIMARY KEY,
            product_name VARCHAR(100),
            category VARCHAR(50),
            price DECIMAL(10,2),
            stock_quantity INT,
            last_updated TIMESTAMP
        );
        ```
    - Insert sample product data:
        ```sql
        INSERT INTO products VALUES
        (1, 'Laptop Pro', 'Electronics', 1299.99, 50, '2024-01-15 10:00:00'),
        (2, 'Desk Chair', 'Furniture', 199.99, 30, '2024-01-16 11:30:00'),
        (3, 'Coffee Maker', 'Appliances', 79.99, 100, '2024-01-14 09:15:00'),
        (4, 'Gaming Mouse', 'Electronics', 59.99, 200, '2024-01-17 14:20:00'),
        (5, 'Bookshelf', 'Furniture', 149.99, 25, '2024-01-13 16:45:00');
        ```
    - FIELD: Order products by category in custom order
        ```sql
        SELECT *, FIELD(category, 'Electronics', 'Appliances', 'Furniture') AS category_order
        FROM products
        ORDER BY FIELD(category, 'Electronics', 'Appliances', 'Furniture') DESC;
        -- Each row gets a number like:
        -- Electronics → 1
        -- Appliances → 2
        -- Furniture → 3
        ```

## 3. Numeric & Mathematical Functions

- Used for calculations and data rounding.
    - Create and use database for numeric function examples:
        ```sql
        CREATE DATABASE NumericFunctionsDB;
        USE NumericFunctionsDB;
        ```
    - Create numbers table:
        ```sql
        CREATE TABLE numbers (
            id INT AUTO_INCREMENT PRIMARY KEY,
            num_value DECIMAL(10,5)
        );
        ```
    - Insert sample numbers:
        ```sql
        INSERT INTO numbers (num_value) VALUES
        (25.6789),
        (-17.5432),
        (100.999),
        (-0.4567),
        (9.5),
        (1234.56789),
        (0);
        ```
    - Basic display of all values:
        ```sql
        SELECT * FROM numbers;
        ```
    - `ABS()`: Returns the absolute (positive) value of a number.
        - Example:
            ```sql
            SELECT num_value, ABS(num_value) AS absolute_value FROM numbers;
            ```
    - `CEIL()` & `FLOOR()`: `CEIL()` rounds up to the nearest integer, `FLOOR()` rounds down.
        - Example:
            ```sql
            SELECT num_value, CEIL(num_value) AS rounded_up, FLOOR(num_value) AS rounded_down FROM numbers;
            ```
    - `ROUND(val, decimals)`: Rounds a number to a specified decimal place.
        - Example:
            ```sql
            SELECT num_value, ROUND(num_value, 2) AS rounded_2_decimals FROM numbers;
            ```
    - `TRUNCATE(val, decimals)`: Cuts off the digits after the specified decimal point without rounding.
        - Example:
            ```sql
            SELECT num_value, TRUNCATE(num_value, 2) AS truncated_2_decimals FROM numbers;
            ```
    - `POWER(base, exp)`: Calculates the power of a number.
        - Example:
            ```sql
            SELECT num_value, POWER(num_value, 2) AS squared FROM numbers;
            ```
    - `SQRT()`: Returns the square root.
        - Example:
            ```sql
            SELECT num_value, SQRT(ABS(num_value)) AS sqrt_value FROM numbers;
            ```
    - `MOD()`: Returns the remainder of a division.
        - Example:
            ```sql
            SELECT num_value, MOD(num_value, 3) AS remainder FROM numbers;
            ```
    - `EXP()`: Returns the value of e (Euler's number, approximately 2.71828) raised to the power of a specified number. 
        - Note: You can give a maximum approximate value of 709 to the `EXP()` so that it does not overflow Double.
        - Example:
            ```sql
            SELECT
                num_value,
                CASE
                    WHEN num_value > 709 THEN 'Value too large for EXP()'
                    ELSE EXP(num_value)
                END AS exp_value
            FROM numbers;
            ```
    - `LOG()`: Returns the logarithm of a number. It can be used in 2 ways:
        - `LOG(B, X)`: Returns the logarithm of X to the specified base B.
        - `LOG10(X)`: Calculates the base-10 logarithm of X.
        - `LOG(X)`: Returns the natural logarithm (base e) of X.
        - `LOG2(X)`: Calculates the base-2 logarithm of X.
        - `LN(X)`: An alternative way to calculate the natural logarithm (equivalent to LOG(X)).
        - Example:
            ```sql
            SELECT num_value,
                   LOG(2, ABS(num_value) + 1) AS log_base2,
                   LOG10(ABS(num_value) + 1) AS log_base10
            FROM numbers;
            ```
    - Trigonometry: Includes `SIN()`, `COS()`, `TAN()` and `PI()`.
        - Example:
            ```sql
            SELECT num_value,
                   SIN(num_value) AS sin_value,
                   COS(num_value) AS cos_value,
                   TAN(num_value) AS tan_value
            FROM numbers;
            ```
        - Pi constant and angle conversions:
            ```sql
            SELECT PI() AS pi_value;
            SELECT num_value,
                   RADIANS(num_value) AS radians_value,
                   DEGREES(num_value) AS degrees_value
            FROM numbers;
            ```
    - Bitwise operations: Used to perform operations directly on the individual bits of binary data.
        - Example:
            ```sql
            SELECT BIT_AND(num_value) FROM numbers;
            SELECT BIT_OR(num_value) FROM numbers;
            SELECT BIT_XOR(num_value) FROM numbers;
            ```

## 4. Date & Time Functions

- Essential for handling temporal data and reports.
    - Date and time data types:
        - `DATE`         YYYY-MM-DD           Stores only date without time
        - `DATETIME`     YYYY-MM-DD HH:MI:SS  Stores date and time
        - `TIMESTAMP`    YYYY-MM-DD HH:MI:SS  Stores date/time with automatic UTC conversion
        - `TIME`         HH:MI:SS             Stores only time
        - `YEAR`         YYYY                 Stores only a four-digit year
    - Date function examples with a database:
        ```sql
        CREATE DATABASE DateExamplesDB;
        USE DateExamplesDB;
        CREATE TABLE orders (
            order_id INT AUTO_INCREMENT PRIMARY KEY,
            customer_name VARCHAR(100),
            order_date DATETIME
        );
        INSERT INTO orders (customer_name, order_date) VALUES
        ('Alice', '2025-03-01 10:15:00'),
        ('Bob', '2025-03-02 14:45:30'),
        ('Charlie', '2025-03-03 09:30:15'),
        ('Akshay', '2024-03-01 10:15:00');
        ```
    - `NOW()`: Returns the current date and time.
        - Example:
            ```sql
            SELECT NOW() AS current_datetime;
            ```
    - `CURDATE()` & `CURTIME()`: Returns only the current date or current time, respectively.
        - Example:
            ```sql
            SELECT CURDATE() AS current_date;
            SELECT CURTIME() AS current_time;
            ```
    - Extraction: `YEAR()`, `MONTH()`, `DAY()`, `HOUR()`, `MINUTE()` and `SECOND()` extract specific parts of a timestamp.
        - Example:
            ```sql
            SELECT YEAR(NOW()) AS current_year;
            SELECT MONTH(NOW()) AS current_month;
            SELECT DAY(NOW()) AS current_day;
            SELECT HOUR(NOW()) AS current_hour;
            SELECT MINUTE(NOW()) AS current_minute;
            SELECT SECOND(NOW()) AS current_second;
            ```
    - `DATE_FORMAT(date, format)`: Formats a date using patterns (e.g., %W for weekday name).
        - Example:
            ```sql
            SELECT DATE_FORMAT('2026-03-30', '%W, %M %e, %Y') AS formatted_date_long;  -- "Monday, March 30, 2026"
            SELECT DATE_FORMAT('2026-03-30', '%e/%m/%Y') AS formatted_date_short;  -- "30/03/2026"
            ```
    - `DATE_ADD()` & `DATE_SUB()`: Adds or subtracts intervals (days, months, years) from a date.
        - Example:
            ```sql
            SELECT DATE_ADD('2026-03-30', INTERVAL 7 MONTH) AS date_plus_7_months;
            SELECT DATE_SUB('2026-03-30', INTERVAL 7 MONTH) AS date_minus_7_months;
            ```
    - `DATEDIFF(date1, date2)`: Returns the number of days between two dates.
        - Example:
            ```sql
            SELECT DATEDIFF('2026-03-30', '2024-03-03') AS days_between;
            ```
    - UNIX Timestamp: An integer value representing the number of seconds that have elapsed since the Unix Epoch (seconds since January 1, 1970, at 00:00:00 UTC)
        - Example:
            ```sql
            SELECT UNIX_TIMESTAMP('2026-03-03') AS UNIX_TIME;
            SELECT FROM_UNIXTIME(1741392000) AS readable_date;
            ```
    - Querying orders in the last 7 days:
        ```sql
        SELECT * FROM orders WHERE order_date >= DATE_SUB(NOW() , INTERVAL 7 DAY);
        ```

## 5. Aggregate Functions

- Used to perform calculations on multiple rows of data and return a single summarized value.
    - Create and use database for aggregate function examples:
        ```sql
        CREATE DATABASE CompanyDB2;
        USE CompanyDB2;
        CREATE TABLE employees (
            id INT AUTO_INCREMENT PRIMARY KEY,
            name VARCHAR(50),
            department VARCHAR(50),
            salary DECIMAL(10,2),
            hire_date DATE
        );
        INSERT INTO employees (name, department, salary, hire_date) VALUES
        ('Alice', 'HR', 50000, '2018-06-23'),
        ('Bob', 'IT', 70000, '2019-08-01'),
        ('Charlie', 'Finance', 80000, '2017-04-15'),
        ('David', 'HR', 55000, '2020-11-30'),
        ('Eve', 'IT', 75000, '2021-01-25'),
        ('Frank', 'Finance', 72000, '2019-07-10'),
        ('Grace', 'IT', 68000, '2018-09-22'),
        ('Hank', 'Finance', 90000, '2016-12-05'),
        ('Ivy', 'HR', 53000, '2022-03-19'),
        ('Jack', 'IT', 72000, '2017-05-12');
        ```
    - `COUNT()`: Returns the number of rows.
        - Example: Count employees in HR department
            ```sql
            SELECT COUNT(*) AS hr_employee_count FROM employees WHERE department='HR';
            ```
    - `SUM()`: Adds up all values in a column.
        - Example: Sum of salaries in HR department
            ```sql
            SELECT SUM(salary) AS total_hr_salary FROM employees WHERE department='HR';
            ```
    - `AVG()`: Calculates the average value.
        - Example: Average salary in HR department
            ```sql
            SELECT AVG(salary) AS avg_hr_salary FROM employees WHERE department='HR';
            ```
    - `MIN()` & `MAX()`: Finds the smallest and largest values in a set.
        - Example: Minimum salary in HR department
            ```sql
            SELECT MIN(salary) AS min_hr_salary FROM employees WHERE department='HR';
            ```
        - Example: Maximum salary in HR department
            ```sql
            SELECT MAX(salary) AS max_hr_salary FROM employees WHERE department='HR';
            ```
    - Comprehensive statistics for all employees:
        ```sql
        SELECT
            COUNT(*) AS num_employees,
            SUM(salary) AS total_salary,
            AVG(salary) AS average_salary,
            MIN(salary) AS lowest_salary,
            MAX(salary) AS highest_salary
        FROM employees;
        ```
    - Group by department to get statistics per department:
        ```sql
        SELECT
            department,
            COUNT(*) AS employee_count,
            SUM(salary) AS department_total_salary,
            ROUND(AVG(salary), 2) AS department_avg_salary,
            MIN(salary) AS department_min_salary,
            MAX(salary) AS department_max_salary
        FROM employees
        GROUP BY department
        ORDER BY department_avg_salary DESC;
        ```