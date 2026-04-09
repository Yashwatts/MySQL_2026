## SQL Stored Procedures and Functions

Stored procedures and user-defined functions let you package SQL logic inside the database. They are useful for reuse, consistency, security and reducing application-side repetition.


## 1. What Are Stored Procedures?

- A stored procedure is a named block of SQL statements saved in the database.
- It can accept parameters, perform multiple actions and optionally return output values.
- It is called explicitly using `CALL`.

Think of a stored procedure as:
- a reusable database program
- a way to bundle multiple SQL steps into one unit
- a good fit for business workflows like order placement, reporting, auditing, or data maintenance

Basic idea:
```sql
CALL get_customer_orders(101);
```


## 2. What Are User-Defined Functions?

- A user-defined function (UDF) is a stored routine that returns a single value.
- It is designed to be used inside SQL expressions.
- Functions are often used for calculations, formatting, validation, or deriving values.

Basic idea:
```sql
SELECT calculate_tax(2500);
```


## 3. Stored Procedures vs Functions

### Stored Procedure

- Can perform multiple SQL statements.
- Can return zero, one, or many result sets.
- Can use `IN`, `OUT` and `INOUT` parameters.
- Called with `CALL`.
- Best for workflows and operational tasks.

### Function

- Must return a single value.
- Can be used in `SELECT`, `WHERE`, `ORDER BY` and expressions.
- Commonly used for calculations and transformations.
- Best for reusable scalar logic.

Comparison example:
```sql
CALL get_employee_by_department(10);

SELECT calculate_bonus(salary) FROM employees;
```


## 4. Procedure vs Function Quick Comparison

| Feature | Stored Procedure | Function |
|---|---|---|
| Called with | `CALL` | Used inside `SELECT` or expressions |
| Return value | Optional result sets / OUT params | Must return one value |
| Parameters | `IN`, `OUT`, `INOUT` | Usually input parameters only |
| Best for | Workflows, transactions, batch tasks | Calculations, transformations |
| Usable in SQL expressions | No | Yes |
| Can return multiple rows | Yes | No |


## 5. CALL vs SELECT Difference

- `CALL` executes a stored procedure.
- `SELECT` evaluates a function or runs a normal query.
- Procedures are invoked as standalone operations.
- Functions are embedded inside SQL expressions.

Examples:
```sql
CALL get_employees_by_dept(10);

SELECT calculate_gst(price) AS gst_amount
FROM products;
```

Important:
- You do not use `SELECT` to execute a procedure.
- You do not use `CALL` to execute a function.


## 6. Creating Stored Procedures

Basic syntax:
```sql
DELIMITER $$

CREATE PROCEDURE procedure_name(IN param1 INT)
BEGIN
    SELECT *
    FROM employees
    WHERE department_id = param1;
END $$

DELIMITER ;
```

Example:
```sql
DELIMITER $$

CREATE PROCEDURE get_employees_by_dept(IN dept_id INT)
BEGIN
    SELECT emp_id, full_name, email, salary
    FROM employees
    WHERE department_id = dept_id;
END $$

DELIMITER ;
```

Call it:
```sql
CALL get_employees_by_dept(10);
```


## 7. Creating User-Defined Functions

Basic syntax:
```sql
DELIMITER $$

CREATE FUNCTION function_name(param1 INT)
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    DECLARE result DECIMAL(10,2);
    SET result = param1 * 1.18;
    RETURN result;
END $$

DELIMITER ;
```

Example:
```sql
DELIMITER $$

CREATE FUNCTION calculate_gst(amount DECIMAL(10,2))
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    RETURN amount * 0.18;
END $$

DELIMITER ;
```

Use it in a query:
```sql
SELECT product_name, price, calculate_gst(price) AS gst_amount
FROM products;
```


## 8. Deterministic vs Non-Deterministic Functions

- Deterministic function:
    - For the same input, it always returns the same output.
    - Example: tax calculation, age from date of birth at a fixed point in time, formatting.

- Non-deterministic function:
    - Same input can produce different output because it depends on current time, random values, or database state.
    - Example: `NOW()`, `RAND()`, `UUID()`.

Examples:
```sql
CREATE FUNCTION add_tax(amount DECIMAL(10,2))
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
        RETURN amount * 1.18;
END;
```

```sql
CREATE FUNCTION current_stamp()
RETURNS DATETIME
NOT DETERMINISTIC
BEGIN
        RETURN NOW();
END;
```

Why this matters:
- MySQL asks you to declare function determinism so it can reason about usage and safety.
- Deterministic functions are more predictable for replication and optimization.


## 9. Restrictions on Functions

Function restrictions are a common interview trap.

Important rules and cautions:
- A function must return exactly one value.
- A function should not return result sets like a procedure.
- Functions are not meant for general multi-step workflows.
- Functions should avoid side-effect-heavy logic.
- Using a function to modify tables is restricted and generally not appropriate for SQL expression use.
- Functions used inside `WHERE` may hurt index usage if they wrap indexed columns.

Typical safe use cases:
- calculations
- formatting
- derived values
- small validation helpers


## 10. Parameters: IN, OUT, INOUT

Parameters allow procedures to receive input and return values.

### 6.1 IN Parameter

- Default direction for input.
- Value is passed into the procedure.
- Inside the procedure, it acts like a read-only input variable.

Example:
```sql
DELIMITER $$

CREATE PROCEDURE get_orders_by_customer(IN p_customer_id INT)
BEGIN
    SELECT order_id, order_date, total_amount
    FROM orders
    WHERE customer_id = p_customer_id;
END $$

DELIMITER ;
```

Call:
```sql
CALL get_orders_by_customer(101);
```

### 6.2 OUT Parameter

- Used to return a value from a procedure.
- Caller passes a variable to receive the result.

Example:
```sql
DELIMITER $$

CREATE PROCEDURE get_total_orders(OUT p_total INT)
BEGIN
    SELECT COUNT(*) INTO p_total
    FROM orders;
END $$

DELIMITER ;
```

Call:
```sql
SET @total_orders = 0;
CALL get_total_orders(@total_orders);
SELECT @total_orders;
```

### 6.3 INOUT Parameter

- Input goes in, modified value comes out.
- Useful when procedure needs to read and change the same variable.

Example:
```sql
DELIMITER $$

CREATE PROCEDURE apply_bonus(INOUT p_salary DECIMAL(10,2))
BEGIN
    SET p_salary = p_salary + (p_salary * 0.10);
END $$

DELIMITER ;
```

Call:
```sql
SET @salary = 50000;
CALL apply_bonus(@salary);
SELECT @salary;
```


## 11. Use Cases for Stored Procedures

Stored procedures are especially useful when:
- multiple SQL statements must run together
- business logic should live close to data
- repeated operations need standardization
- auditing and logging are required
- batch processing is needed
- application code should stay thinner

Examples:
- order checkout
- payment posting
- inventory update
- monthly reporting
- archiving old records
- centralized validation


## 12. Use Cases for Functions

Functions are useful when:
- you need a reusable calculation
- you want to transform data in queries
- logic is small and expression-oriented
- a value must be returned inside a `SELECT`

Examples:
- tax calculation
- age calculation
- string cleanup
- status formatting
- date conversion


## 13. Example: Procedure for Order Processing

```sql
DELIMITER $$

CREATE PROCEDURE place_order(
    IN p_customer_id INT,
    IN p_product_id INT,
    IN p_qty INT,
    OUT p_order_id INT
)
BEGIN
    DECLARE v_price DECIMAL(10,2);
    DECLARE v_total DECIMAL(10,2);

    SELECT price INTO v_price
    FROM products
    WHERE product_id = p_product_id;

    SET v_total = v_price * p_qty;

    INSERT INTO orders(customer_id, order_date, total_amount)
    VALUES (p_customer_id, NOW(), v_total);

    SET p_order_id = LAST_INSERT_ID();

    INSERT INTO order_items(order_id, product_id, qty, price)
    VALUES (p_order_id, p_product_id, p_qty, v_price);
END $$

DELIMITER ;
```

Call:
```sql
SET @order_id = 0;
CALL place_order(101, 5, 3, @order_id);
SELECT @order_id;
```


## 14. Example: Function for Age Calculation

```sql
DELIMITER $$

CREATE FUNCTION get_age(dob DATE)
RETURNS INT
DETERMINISTIC
BEGIN
    RETURN TIMESTAMPDIFF(YEAR, dob, CURDATE());
END $$

DELIMITER ;
```

Use it:
```sql
SELECT full_name, dob, get_age(dob) AS age
FROM customers;
```


## 15. Variables Inside Procedures and Functions

### Local Variables

Declare with `DECLARE` inside routine body.

```sql
DECLARE total_salary DECIMAL(10,2);
DECLARE counter INT DEFAULT 0;
```

### Session Variables

Session variables are prefixed with `@`.

```sql
SET @value = 100;
SELECT @value;
```

Use local variables when possible because they stay inside the routine scope.


## 16. Control Flow in Stored Routines

MySQL supports control structures such as:
- `IF ... THEN ... ELSE`
- `CASE`
- `LOOP`
- `WHILE`
- `REPEAT`
- handlers for exceptions

Example:
```sql
DELIMITER $$

CREATE PROCEDURE check_order_status(IN p_order_id INT)
BEGIN
    DECLARE v_status VARCHAR(20);

    SELECT status INTO v_status
    FROM orders
    WHERE order_id = p_order_id;

    IF v_status = 'PAID' THEN
        SELECT 'Order is paid' AS message;
    ELSE
        SELECT 'Order is not paid' AS message;
    END IF;
END $$

DELIMITER ;
```


## 17. Error Handling with HANDLER

You can handle errors inside procedures.

Example:
```sql
DELIMITER $$

CREATE PROCEDURE safe_insert_customer(
    IN p_name VARCHAR(100),
    IN p_email VARCHAR(150)
)
BEGIN
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        SELECT 'Insert failed and transaction rolled back' AS message;
    END;

    START TRANSACTION;

    INSERT INTO customers(full_name, email)
    VALUES (p_name, p_email);

    COMMIT;
END $$

DELIMITER ;
```


## 18. Transaction Use in Procedures

Stored procedures often work with transactions.

Example pattern:
```sql
DELIMITER $$

CREATE PROCEDURE transfer_money(
    IN p_from_account INT,
    IN p_to_account INT,
    IN p_amount DECIMAL(10,2)
)
BEGIN
    START TRANSACTION;

    UPDATE accounts
    SET balance = balance - p_amount
    WHERE account_id = p_from_account;

    UPDATE accounts
    SET balance = balance + p_amount
    WHERE account_id = p_to_account;

    COMMIT;
END $$

DELIMITER ;
```

If validation fails, use `ROLLBACK`.


## 19. Security and Maintenance Benefits

- Logic is centralized in the database.
- Users can be granted permission to call routines instead of direct table access.
- Repeated SQL is reduced in application code.
- Business rules become easier to standardize.

Example permission model:
```sql
GRANT EXECUTE ON PROCEDURE place_order TO 'app_user'@'localhost';
GRANT EXECUTE ON FUNCTION get_age TO 'app_user'@'localhost';
```


## 20. Important MySQL Notes

- Use `DELIMITER` when creating routines containing semicolons.
- Functions must return a value and cannot directly return result sets like procedures.
- A function used in SQL should be deterministic when appropriate.
- Avoid writing functions that do heavy transactional work.
- Keep routine names descriptive and consistent.


## 21. Stored Procedure and Function Limitations

- Overusing routines can hide logic and make debugging harder.
- Complex routines can be difficult to version and maintain.
- Performance depends on how the routine is written and indexed tables.
- Functions used in `WHERE` clauses can sometimes reduce index use if written poorly.


## 22. Performance Warning: Functions in WHERE

- Putting a function around an indexed column can stop MySQL from using the index efficiently.
- This can force a full scan or reduce optimizer options.

Bad example:
```sql
SELECT *
FROM employees
WHERE YEAR(hire_date) = 2026;
```

Better example:
```sql
SELECT *
FROM employees
WHERE hire_date >= '2026-01-01'
  AND hire_date < '2027-01-01';
```

Rule:
- Keep indexed columns on the left side of comparisons whenever possible.


## 23. Cursors (Very Important)

Cursors let a stored procedure process query results row by row.

- Use cursors when set-based SQL is not enough.
- They are slower than set-based queries, but sometimes necessary for procedural logic.

Cursor lifecycle:
- Declare cursor
- Open cursor
- Fetch rows one by one
- Close cursor

Example:
```sql
DELIMITER $$

CREATE PROCEDURE process_employee_rows()
BEGIN
    DECLARE done INT DEFAULT 0;
    DECLARE v_emp_id INT;
    DECLARE v_salary DECIMAL(10,2);

    DECLARE emp_cursor CURSOR FOR
        SELECT emp_id, salary FROM employees;

    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = 1;

    OPEN emp_cursor;

    read_loop: LOOP
        FETCH emp_cursor INTO v_emp_id, v_salary;

        IF done = 1 THEN
            LEAVE read_loop;
        END IF;

        -- Example row-by-row logic
        INSERT INTO salary_audit(emp_id, salary_checked, checked_at)
        VALUES (v_emp_id, v_salary, NOW());
    END LOOP;

    CLOSE emp_cursor;
END $$

DELIMITER ;
```


## 24. Dynamic SQL in Procedures

Dynamic SQL means building SQL text at runtime and executing it.

Why use it:
- table name changes
- column selection changes
- flexible reporting logic

Example:
```sql
DELIMITER $$

CREATE PROCEDURE count_rows_dynamic(IN p_table_name VARCHAR(64))
BEGIN
    SET @sql = CONCAT('SELECT COUNT(*) AS row_count FROM ', p_table_name);
    PREPARE stmt FROM @sql;
    EXECUTE stmt;
    DEALLOCATE PREPARE stmt;
END $$

DELIMITER ;
```

Call:
```sql
CALL count_rows_dynamic('employees');
```

Important caution:
- Dynamic SQL must be validated carefully to avoid SQL injection.
- Prefer whitelisting allowed table or column names.


## 25. Best Practices

- Use procedures for workflows and multi-step operations.
- Use functions for small reusable calculations.
- Keep routine logic focused and readable.
- Handle errors explicitly.
- Test transaction behavior carefully.
- Document input and output parameters.
- Avoid unnecessary logic inside functions that are called frequently.


## 26. Quick Interview Notes

- Stored procedures: named SQL blocks executed with `CALL`.
- Functions: return one value and can be used inside expressions.
- `IN` is input-only, `OUT` returns a value, `INOUT` does both.
- Procedures are better for business workflows.
- Functions are better for calculations and transformations.
- Deterministic functions always return the same output for the same input.
- Functions in `WHERE` can harm index usage.
- Cursors process rows one at a time and should be used sparingly.
- Dynamic SQL is powerful but must be handled carefully.


## 27. Final Summary

- Stored procedures help package multi-step database logic.
- Functions help reuse scalar logic inside SQL.
- Parameters (`IN`, `OUT`, `INOUT`) control data flow.
- Use procedures for process automation and functions for expressions.
- Proper design improves reuse, security and consistency.
