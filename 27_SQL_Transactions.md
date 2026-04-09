## SQL Transactions and ACID Properties

Transactions are the foundation of reliable database systems. They make sure related SQL operations are treated as one logical unit, so your data stays correct even when failures happen.


## 1. What Is a Transaction?

- A transaction is a sequence of one or more SQL statements executed as a single unit of work.
- Either all statements succeed, or the whole unit is reversed.
- Transactions are critical in banking, inventory, payments and any multi-step business operation.

Example scenario:
- Transfer money from Account A to Account B.
- If debit succeeds but credit fails, data becomes incorrect.
- Transaction guarantees both changes happen together or none happen.


## 2. Transaction Control Commands

### 2.1 BEGIN / START TRANSACTION

Use this to start an explicit transaction.

```sql
BEGIN;
-- or
START TRANSACTION;
```

### 2.2 COMMIT

Use this to permanently save all changes made in the current transaction.

```sql
COMMIT;
```

### 2.3 ROLLBACK

Use this to undo all uncommitted changes in the current transaction.

```sql
ROLLBACK;
```

### 2.4 Basic Transaction Example

```sql
START TRANSACTION;

UPDATE accounts
SET balance = balance - 1000
WHERE account_id = 1;

UPDATE accounts
SET balance = balance + 1000
WHERE account_id = 2;

COMMIT;
```

If any step fails, rollback:

```sql
START TRANSACTION;

UPDATE accounts
SET balance = balance - 1000
WHERE account_id = 1;

-- Suppose next step fails due to constraint or connection issue
UPDATE accounts
SET balance = balance + 1000
WHERE account_id = 2;

ROLLBACK;
```


## 3. Autocommit in MySQL

- MySQL default mode is `AUTOCOMMIT = 1`.
- That means each statement is committed automatically unless you start an explicit transaction.

Check autocommit:
```sql
SELECT @@autocommit;
```

Disable autocommit for session:
```sql
SET autocommit = 0;
```

Important:
- With `autocommit = 0`, changes remain pending until `COMMIT` or `ROLLBACK`.
- Be careful not to leave long transactions open, because they can increase locks and undo logs.


## 4. ACID Properties

ACID ensures transaction reliability.

### 4.1 Atomicity (All or Nothing)

- A transaction is indivisible.
- Either every statement succeeds or none of them are applied.

Example:
```sql
START TRANSACTION;

UPDATE wallet SET amount = amount - 500 WHERE user_id = 10;
UPDATE wallet SET amount = amount + 500 WHERE user_id = 20;

COMMIT;
```

If second statement fails, `ROLLBACK` ensures first statement is also undone.

### 4.2 Consistency (Valid State to Valid State)

- A transaction must preserve all defined rules and constraints.
- Data moves from one valid state to another valid state.

Examples of consistency checks:
- Primary key and foreign key constraints
- Unique constraints
- Check constraints
- Application business rules

If a transaction violates constraints, commit is rejected and transaction should rollback.

### 4.3 Isolation (Transactions Do Not Interfere Incorrectly)

- Concurrent transactions should not corrupt each other.
- Isolation level controls how much one transaction can see effects of another uncommitted or recently committed transaction.

### 4.4 Durability (Committed Means Permanent)

- Once `COMMIT` succeeds, changes survive crashes/power loss.
- InnoDB ensures this using logs (redo log) and crash recovery.


## 5. Isolation Problems (Why Levels Matter)

### 5.1 Dirty Read

- Transaction T1 reads data modified by T2 before T2 commits.
- If T2 rolls back, T1 read invalid data.

### 5.2 Non-Repeatable Read

- T1 reads same row twice and gets different values because T2 committed an update between reads.

### 5.3 Phantom Read

- T1 runs same range query twice and sees different set of rows because T2 inserted/deleted matching rows.
- MySQL-specific note:
  - In InnoDB `REPEATABLE READ`, plain consistent reads use the same snapshot, so many phantom-like effects are hidden for normal `SELECT`.
  - For locking reads (`SELECT ... FOR UPDATE` / `SELECT ... FOR SHARE`), InnoDB uses next-key locking to block inserts into matching ranges and reduce phantom rows.


## 6. Isolation Levels in MySQL

MySQL (InnoDB) supports four standard isolation levels.

### 6.1 Read Uncommitted

- Lowest isolation, highest concurrency.
- Dirty reads are possible.
- Rarely used in production.

Set level:
```sql
SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
```

### 6.2 Read Committed

- Prevents dirty reads.
- Non-repeatable reads and phantom reads can still happen.
- Common in several database systems.

Set level:
```sql
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

### 6.3 Repeatable Read (MySQL InnoDB Default)

- Prevents dirty reads and non-repeatable reads.
- InnoDB uses MVCC, and with locking reads can reduce phantom issues significantly.
- Default and widely used level in MySQL.

Set level:
```sql
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```

### 6.4 Serializable

- Highest isolation.
- Transactions behave as if executed one by one.
- Strong consistency, but lower concurrency and more locking/waiting.

Set level:
```sql
SET SESSION TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```


## 7. Isolation Level Comparison Table

- Read Uncommitted:
  - Dirty Read: Yes
  - Non-Repeatable Read: Yes
  - Phantom Read: Yes

- Read Committed:
  - Dirty Read: No
  - Non-Repeatable Read: Yes
  - Phantom Read: Yes

- Repeatable Read:
  - Dirty Read: No
  - Non-Repeatable Read: No
  - Phantom Read: By SQL standard it is possible, but InnoDB often prevents it in locking range reads using next-key locks

- Serializable:
  - Dirty Read: No
  - Non-Repeatable Read: No
  - Phantom Read: No


## 8. Practical Session-Based Example

Assume row exists:
```sql
SELECT * FROM products WHERE product_id = 101;
```

### Session A
```sql
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
START TRANSACTION;

SELECT stock FROM products WHERE product_id = 101;
-- returns 25
```

### Session B (runs in parallel)
```sql
START TRANSACTION;
UPDATE products SET stock = 20 WHERE product_id = 101;
COMMIT;
```

### Session A again
```sql
SELECT stock FROM products WHERE product_id = 101;
-- under READ COMMITTED, may now return 20 (non-repeatable read)
COMMIT;
```

Under `REPEATABLE READ`, Session A would keep seeing its original consistent snapshot for plain selects in the same transaction.


## 9. Transaction-Safe Transfer Example (Realistic)

```sql
START TRANSACTION;

-- Lock both rows to avoid race conditions in concurrent transfers
SELECT balance
FROM accounts
WHERE account_id IN (1, 2)
FOR UPDATE;

-- Validate source has sufficient balance in application logic
UPDATE accounts
SET balance = balance - 1000
WHERE account_id = 1;

UPDATE accounts
SET balance = balance + 1000
WHERE account_id = 2;

INSERT INTO transfer_log(from_account, to_account, amount, transfer_time)
VALUES (1, 2, 1000, NOW());

COMMIT;
```

If any validation/check fails:
```sql
ROLLBACK;
```


## 10. SAVEPOINT (Partial Rollback Inside Transaction)

`SAVEPOINT` is useful when you want to undo only part of a transaction.

```sql
START TRANSACTION;

INSERT INTO orders(order_id, customer_id, total_amount)
VALUES (5001, 101, 1500);

SAVEPOINT after_order;

INSERT INTO order_items(order_id, product_id, qty, price)
VALUES (5001, 9999, 2, 750); -- suppose this fails logically

ROLLBACK TO SAVEPOINT after_order;

-- Continue with valid items
INSERT INTO order_items(order_id, product_id, qty, price)
VALUES (5001, 1010, 2, 750);

COMMIT;
```


## 11. Implicit Commit in MySQL (Important)

Some statements cause implicit commit before/after execution, especially many DDL statements.

Examples that typically trigger implicit commit:
- `CREATE TABLE`
- `ALTER TABLE`
- `DROP TABLE`
- `TRUNCATE TABLE`

So avoid mixing DDL and DML in one transaction when you expect full rollback behavior.


## 12. Locking and MVCC (InnoDB Essentials)

- InnoDB uses MVCC (Multi-Version Concurrency Control) for consistent reads.
- Normal `SELECT` usually does not block writers.
- Locking reads:
  - `SELECT ... FOR UPDATE` for exclusive row locks
  - `SELECT ... FOR SHARE` (or `LOCK IN SHARE MODE` in older syntax) for shared read locks

Use locking reads only when needed, because too much locking can reduce concurrency.

### 12.1 Read vs Write Locks

- Read lock (shared lock):
  - Multiple transactions can read same locked row.
  - Blocks writers from modifying that row until lock is released.

- Write lock (exclusive lock):
  - Transaction can read and modify locked row.
  - Blocks other readers (locking reads) and writers from taking conflicting locks.

Practical usage:
- Use `FOR SHARE` when you need stable read before decision logic.
- Use `FOR UPDATE` when you will update/delete those rows.

### 12.2 Gap Locks and Next-Key Locks

- Gap lock:
  - Locks the gap between index records (not just existing row values).
  - Prevents inserts into that gap.

- Next-key lock:
  - Combination of row lock + gap lock.
  - Used by InnoDB to prevent phantom inserts in locking range queries.

Example:
```sql
START TRANSACTION;

SELECT *
FROM orders
WHERE order_id BETWEEN 100 AND 200
FOR UPDATE;
```

InnoDB may apply next-key locks on that index range so concurrent inserts in the locked range wait.

### 12.3 Undo Logs vs Redo Logs

- Undo log: supports rollback and MVCC old-version reads.
- Redo log: guarantees durability by replaying committed changes during crash recovery.


## 13. Deadlock Explanation (Very Important)

Deadlock happens when two (or more) transactions wait on each other forever in a cycle.

Example deadlock pattern:
- T1 locks row A, then requests row B.
- T2 locks row B, then requests row A.
- Both wait indefinitely unless database breaks the cycle.

InnoDB behavior:
- InnoDB detects deadlocks automatically.
- It aborts one transaction (deadlock victim) with an error.
- The other transaction can continue.

Illustrative flow:
```sql
-- Session 1
START TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
-- waits later on account_id = 2

-- Session 2
START TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 2;
-- waits later on account_id = 1
```

Deadlock prevention tips:
- Access tables/rows in a consistent order across all transactions.
- Keep transactions short.
- Index properly so scans lock fewer rows.
- Retry aborted transaction in application logic.


## 14. Read-Only vs Read-Write Transactions

- Read-only transaction:
  - Intended only for reads.
  - Can reduce overhead in some workloads and communicate clear intent.

- Read-write transaction:
  - Default mode when data modifications are expected.

Examples:
```sql
START TRANSACTION READ ONLY;
SELECT * FROM products WHERE category_id = 10;
COMMIT;

START TRANSACTION READ WRITE;
UPDATE products SET price = price * 1.05 WHERE category_id = 10;
COMMIT;
```


## 15. Transaction States

Conceptually, a transaction moves through these states:

- Active:
  - Transaction has started and statements are executing.

- Committed:
  - Transaction finished successfully and changes are permanent.

- Aborted (Rolled Back):
  - Transaction failed or was explicitly rolled back; its changes are undone.

Simple lifecycle:
- `Active -> Committed`
- `Active -> Aborted`


## 16. Best Practices for Transactions

- Keep transactions short.
- Access rows in consistent order across transactions to reduce deadlocks.
- Use appropriate isolation level, not always the strictest one.
- Use `SELECT ... FOR UPDATE` for critical read-modify-write flows.
- Always handle errors with rollback logic in application code.
- Avoid user interaction while transaction is open.
- Monitor deadlocks and long-running transactions.


## 17. Quick Interview Notes

- ACID full form:
  - Atomicity, Consistency, Isolation, Durability
- MySQL InnoDB default isolation level:
  - `REPEATABLE READ`
- Command flow:
  - `BEGIN/START TRANSACTION -> SQL statements -> COMMIT or ROLLBACK`
- Use `SAVEPOINT` for partial undo.
- Durable commit relies on InnoDB logging and recovery.
- Deadlock means cyclic waiting; InnoDB auto-detects and aborts one transaction.
- `REPEATABLE READ` is MySQL InnoDB default and behaves differently from textbook expectations due to MVCC + next-key locking.


## 18. Final Summary

- Transactions ensure reliable multi-step operations.
- `BEGIN`, `COMMIT`, and `ROLLBACK` are core transaction commands.
- ACID guarantees correctness, safety, and persistence of committed data.
- Isolation levels control concurrency vs consistency trade-off.
- Choose isolation level based on business requirements and performance needs.
- Correct locking strategy and deadlock-aware design are essential in real production systems.
