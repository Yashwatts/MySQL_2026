## SQL Triggers

Triggers are special database objects that automatically execute when a specific event happens on a table. They are useful for enforcing rules, auditing changes and keeping related data in sync.


## 1. What Is a Trigger?

- A trigger is a stored program that runs automatically in response to `INSERT`, `UPDATE` or `DELETE` on a table.
- It is attached to a specific table.
- It fires before or after the triggering event.
- Triggers run row by row when a statement affects multiple rows.

Think of a trigger as:
- automatic database logic
- a safety net for data rules
- a way to record or react to changes without application code

Example idea:
- When a new order is inserted, automatically reduce stock.
- When a row is updated, write old and new values into an audit table.
- When a row is deleted, store its details in a history table.


## 2. Why Triggers Are Useful

- Enforce business rules inside the database.
- Maintain audit logs automatically.
- Keep summary or history tables updated.
- Prevent invalid changes.
- Reduce repeated logic in the application layer.


## 2.1 Trigger vs Stored Procedure (Common Interview Question)

- Trigger:
    - Runs automatically when a table event occurs (`INSERT`, `UPDATE`, `DELETE`).
    - Cannot be called directly by name.
    - Best for automatic reactions like audit logging and validation.

- Stored Procedure:
    - Runs only when explicitly called using `CALL`.
    - Can accept `IN`, `OUT`, `INOUT` parameters.
    - Best for controlled workflows and multi-step operations.

Quick example:
```sql
-- Procedure is called explicitly
CALL place_order(101, 5, 2, @new_order_id);

-- Trigger runs automatically after insert (no explicit call)
INSERT INTO orders(customer_id, order_date, total_amount)
VALUES (101, NOW(), 2500);
```


## 3. Trigger Timing and Events

### 3.1 BEFORE Trigger

- Runs before the row is inserted, updated, or deleted.
- Useful when you want to validate or modify values before saving.

### 3.2 AFTER Trigger

- Runs after the row change has happened.
- Useful for audit logging, history tracking and maintaining related tables.

### 3.3 Trigger Events

MySQL supports triggers for these events:
- `INSERT`
- `UPDATE`
- `DELETE`

So the common combinations are:
- `BEFORE INSERT`
- `AFTER INSERT`
- `BEFORE UPDATE`
- `AFTER UPDATE`
- `BEFORE DELETE`
- `AFTER DELETE`

### 3.4 Row-Level vs Statement-Level (MySQL Nuance)

- Row-level trigger:
    - Fires once for each affected row.
    - MySQL supports this using `FOR EACH ROW`.

- Statement-level trigger:
    - Fires once per SQL statement, regardless of row count.
    - MySQL does not support statement-level triggers.

Example:
- If one `UPDATE` statement modifies 100 rows, MySQL trigger executes 100 times.


## 4. Trigger Syntax

Basic syntax:
```sql
DELIMITER $$

CREATE TRIGGER trigger_name
BEFORE INSERT ON table_name
FOR EACH ROW
BEGIN
    -- trigger logic
END $$

DELIMITER ;
```

Important points:
- `FOR EACH ROW` means the trigger fires once for every affected row.
- `NEW` refers to the new row values.
- `OLD` refers to the previous row values.

### 4.1 Trigger Naming Convention

Use clear names so trigger intent is obvious.

Common naming pattern:
- `trg_<table>_<timing>_<event>`

Examples:
- `trg_customers_before_insert`
- `trg_employees_after_update`
- `trg_products_before_delete`

Good naming makes debugging and maintenance much easier.


## 5. NEW and OLD in Triggers

### 5.1 NEW

- Represents the row being inserted or the new values in an update.
- Available in `INSERT` and `UPDATE` triggers.

### 5.2 OLD

- Represents the previous row values.
- Available in `UPDATE` and `DELETE` triggers.

Quick map:
- `INSERT` trigger: `NEW` only
- `UPDATE` trigger: `OLD` and `NEW`
- `DELETE` trigger: `OLD` only


## 6. BEFORE INSERT Trigger

Use case:
- auto-generate values
- validate data
- normalize text

Example: set default formatting before insert.
```sql
DELIMITER $$

CREATE TRIGGER before_customer_insert
BEFORE INSERT ON customers
FOR EACH ROW
BEGIN
    SET NEW.full_name = TRIM(NEW.full_name);
    SET NEW.email = LOWER(NEW.email);
END $$

DELIMITER ;
```

If someone inserts messy data:
```sql
INSERT INTO customers(full_name, email)
VALUES ('  Alice Johnson  ', 'ALICE@EXAMPLE.COM');
```

Trigger transforms it before saving.

Another common use:
- generate a slug
- set a creation timestamp
- validate price or quantity


## 7. AFTER INSERT Trigger

Use case:
- write audit logs
- update summary tables
- create related records

Example: log new sales orders.
```sql
DELIMITER $$

CREATE TRIGGER after_order_insert
AFTER INSERT ON orders
FOR EACH ROW
BEGIN
    INSERT INTO order_audit(order_id, action_type, changed_at)
    VALUES (NEW.order_id, 'INSERT', NOW());
END $$

DELIMITER ;
```

This is useful because the row already exists, so the audit record can safely reference it.


## 8. BEFORE UPDATE Trigger

Use case:
- validate values before saving
- enforce business rules
- prevent invalid changes
- modify values before update

Example: prevent negative salary and trim name.
```sql
DELIMITER $$

CREATE TRIGGER before_employee_update
BEFORE UPDATE ON employees
FOR EACH ROW
BEGIN
    IF NEW.salary < 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Salary cannot be negative';
    END IF;

    SET NEW.full_name = TRIM(NEW.full_name);
END $$

DELIMITER ;
```

Here:
- the trigger rejects bad data using `SIGNAL`
- the trigger also cleans the incoming name


## 9. AFTER UPDATE Trigger

Use case:
- track old and new values
- maintain audit history
- update related summary tables

Example: store update history.
```sql
DELIMITER $$

CREATE TRIGGER after_employee_update
AFTER UPDATE ON employees
FOR EACH ROW
BEGIN
    INSERT INTO employee_history(emp_id, old_salary, new_salary, changed_at)
    VALUES (OLD.emp_id, OLD.salary, NEW.salary, NOW());
END $$

DELIMITER ;
```

This is very common in real systems where you need a record of what changed.


## 10. BEFORE DELETE Trigger

Use case:
- prevent deletion of protected rows
- move data to archive/history first
- enforce business restrictions

Example: block deleting active admins.
```sql
DELIMITER $$

CREATE TRIGGER before_user_delete
BEFORE DELETE ON users
FOR EACH ROW
BEGIN
    IF OLD.role = 'admin' THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Admin users cannot be deleted';
    END IF;
END $$

DELIMITER ;
```


## 11. AFTER DELETE Trigger

Use case:
- archive deleted records
- write deletion audit logs
- maintain dependent tables

Example: save deleted row into archive.
```sql
DELIMITER $$

CREATE TRIGGER after_product_delete
AFTER DELETE ON products
FOR EACH ROW
BEGIN
    INSERT INTO deleted_products(product_id, product_name, deleted_at)
    VALUES (OLD.product_id, OLD.product_name, NOW());
END $$

DELIMITER ;
```


## 12. Real-World Use Cases for Triggers

Triggers are commonly used for:
- audit trails
- automatic timestamp maintenance
- data validation
- inventory adjustment
- soft delete support
- archive/history tables
- maintaining aggregated totals
- enforcing custom business rules

Examples:
- When an order is inserted, reduce product stock.
- When a salary is updated, write old and new salary to history.
- When a row is deleted, store it in an archive table.
- When a customer name is inserted, clean formatting automatically.


## 13. Example: Inventory Update Trigger

When a new order item is inserted, reduce stock automatically.

```sql
DELIMITER $$

CREATE TRIGGER after_order_item_insert
AFTER INSERT ON order_items
FOR EACH ROW
BEGIN
    UPDATE products
    SET stock_quantity = stock_quantity - NEW.qty
    WHERE product_id = NEW.product_id;
END $$

DELIMITER ;
```

This keeps inventory consistent without relying only on the application.


## 14. Example: Audit Trigger

Create an audit table:
```sql
CREATE TABLE employee_audit (
    audit_id INT AUTO_INCREMENT PRIMARY KEY,
    emp_id INT,
    action_type VARCHAR(20),
    old_salary DECIMAL(10,2),
    new_salary DECIMAL(10,2),
    changed_at DATETIME
);
```

Trigger:
```sql
DELIMITER $$

CREATE TRIGGER after_employee_salary_update
AFTER UPDATE ON employees
FOR EACH ROW
BEGIN
    IF OLD.salary <> NEW.salary THEN
        INSERT INTO employee_audit(emp_id, action_type, old_salary, new_salary, changed_at)
        VALUES (OLD.emp_id, 'UPDATE', OLD.salary, NEW.salary, NOW());
    END IF;
END $$

DELIMITER ;
```


## 15. Example: Prevent Invalid Data

```sql
DELIMITER $$

CREATE TRIGGER before_product_insert
BEFORE INSERT ON products
FOR EACH ROW
BEGIN
    IF NEW.price <= 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Price must be greater than zero';
    END IF;
END $$

DELIMITER ;
```

This is a clean way to enforce business rules inside the database.


## 16. How to View Triggers

Show triggers in the current database:
```sql
SHOW TRIGGERS;
```

Show trigger definition:
```sql
SHOW CREATE TRIGGER after_employee_update;
```


## 17. How to Drop a Trigger

```sql
DROP TRIGGER IF EXISTS after_employee_update;
```


## 18. Trigger Order and Multiple Triggers

- MySQL can have multiple triggers for the same table and event timing (supported in modern MySQL versions).
- Trigger execution order can matter.
- In newer MySQL versions, `FOLLOWS` and `PRECEDES` can help control order.

Important MySQL nuance:
- Older MySQL versions allowed only one trigger per table per timing/event.
- In current MySQL versions, multiple triggers are allowed, but order must be managed carefully.

Conceptual example:
```sql
CREATE TRIGGER trg1 BEFORE INSERT ON products
FOR EACH ROW
BEGIN
    SET NEW.created_at = NOW();
END;
```

If another trigger exists for the same event, MySQL can determine execution order based on trigger creation or explicit ordering support.


## 19. Important Restrictions and Limits

- Triggers are tied to one table.
- They fire per row, so large bulk operations can make them expensive.
- Triggers should stay small and focused.
- Overusing triggers can make debugging harder.
- Avoid putting too much business logic in triggers if application logic is clearer.
- You cannot use `COMMIT` or `ROLLBACK` inside a trigger.
- Be careful with recursion-like side effects through related table updates.

### 19.1 No COMMIT / ROLLBACK Inside Trigger

- Trigger execution is part of the same transaction as the statement that fired it.
- Transaction boundaries are controlled outside the trigger.
- So `COMMIT` and `ROLLBACK` are not allowed inside trigger bodies.

### 19.2 Recursion / Infinite Loop Risk

- Direct self-recursion is restricted, but indirect loops can still happen through chained table updates.
- Example risk:
    - Trigger on table A updates table B
    - Trigger on table B updates table A
    - This can cause repeated firing, lock contention, or failures.

Prevention tips:
- Keep trigger logic minimal.
- Avoid circular update design.
- Use clear guard conditions in trigger logic.


## 20. Performance Considerations

Triggers can slow down writes because they run automatically for every affected row.

Performance issues happen when:
- the trigger does extra joins
- the trigger writes to multiple tables
- the trigger performs heavy validation on large data sets
- the triggering statement affects many rows

Best practice:
- keep trigger logic short
- index columns used inside trigger queries
- avoid unnecessary work inside triggers


## 20.1 When NOT to Use Triggers

Avoid triggers when:
- logic is complex and easier to manage in application or stored procedures
- behavior must be very explicit and visible to all developers
- high-throughput bulk writes are performance-critical
- operations need easy step-by-step debugging
- business rules depend on external services/APIs

Use triggers for compact, data-close rules. Use application/procedure logic for large orchestration workflows.


## 21. BEFORE vs AFTER Triggers

### BEFORE Trigger

- Best when you need to validate or modify incoming values.
- Useful for cleanup, normalization and rejection of bad data.

### AFTER Trigger

- Best when you need to log or react to data that already changed.
- Useful for audit records, history tables and summary updates.

Rule of thumb:
- Use `BEFORE` if you must influence the row being saved.
- Use `AFTER` if you need the final saved row.


## 22. INSERT / UPDATE / DELETE Trigger Summary

### INSERT triggers
- `BEFORE INSERT`: validate or adjust new values.
- `AFTER INSERT`: log or update related tables.

### UPDATE triggers
- `BEFORE UPDATE`: validate or modify new values.
- `AFTER UPDATE`: capture old/new values or update history.

### DELETE triggers
- `BEFORE DELETE`: block or validate deletion.
- `AFTER DELETE`: archive or log deleted rows.


## 23. Best Practices

- Use triggers only for logic that truly belongs close to the data.
- Keep them simple and predictable.
- Use them for auditing, validation and consistency rules.
- Document all triggers clearly.
- Test bulk operations carefully.
- Prefer set-based SQL when triggers are not necessary.
- Use `SIGNAL` to stop invalid actions cleanly.


## 24. Quick Interview Notes

- A trigger runs automatically in response to `INSERT`, `UPDATE`, or `DELETE`.
- `BEFORE` triggers can change or validate incoming data.
- `AFTER` triggers are good for logging and related updates.
- `NEW` is the incoming row, `OLD` is the previous row.
- Triggers are powerful but can hurt performance if overused.
- MySQL supports row-level triggers (`FOR EACH ROW`), not statement-level triggers.
- Triggers and procedures are different: trigger is automatic, procedure is called explicitly.
- `COMMIT` / `ROLLBACK` cannot be written inside trigger body.


## 25. Final Summary

- Triggers automate database reactions to data changes.
- They are commonly used for auditing, validation, inventory updates and history tracking.
- `BEFORE` and `AFTER` determine when the trigger runs.
- `INSERT`, `UPDATE` and `DELETE` determine what event causes it.
- Good trigger design improves data integrity but poor design can make systems harder to debug and maintain.
