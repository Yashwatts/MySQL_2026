# Database Level Commands

This file focuses on the fundamental operations required to create, view, select and delete databases in MySQL.

## 1. Creating a Database

To start storing data (e.g., for a school or e-commerce site), you must first create a dedicated database container.

- Command: CREATE DATABASE database_name;
- Example: CREATE DATABASE school_db;
- Naming Rules: You can use characters, numbers and underscores.

## 2. Viewing Available Databases

You can see a list of all databases currently residing on your MySQL server.

- Command: SHOW DATABASES;
- NOTE: This will show both your custom databases and system-generated ones (like sys or performance_schema and will also show school_db)

## 3. Selecting a Database

Before you can create tables or insert data, you must tell MySQL which database you want to work in.

- Command: USE database_name;
- Example: USE school_db;
- Analogy: Just like navigating to a specific folder to play a movie, you must "enter" the database first.

## 4. Checking the Current Database

If you are unsure which database is currently active, use this function.

- Command: SELECT DATABASE();
- Result: It returns the name of the database you are currently using.

## 5. Deleting (Dropping) a Database

To permanently remove a database and all its contents.

- Command: DROP DATABASE database_name;
- Example: DROP DATABASE school_db;
- Warning: This action is irreversible and removes all associated internal metadata as well.

## Key Tips & Syntax

- Case Insensitivity: In MySQL (especially on Windows), keywords and database names are case-insensitive. CREATE is the same as create or creATe.
- The Semicolon(;): Use a semicolon to terminate each command, especially when running multiple SQL statements at once, to tell the server where one ends and the next begins.
- MySQL Workbench: To execute a command in Workbench, you can highlight the specific line and click the Run (lightning Bolt) icon.

## Final Code Block

```sql
creAte database school_db;
show databases;
UsE SCHooL_DB;
SELECT DATABASE();
drop database school_db;
```