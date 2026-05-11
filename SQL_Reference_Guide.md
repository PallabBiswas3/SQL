# 🗄️ SQL Complete Reference Guide
> A comprehensive cheat sheet from basic to advanced — perfect for solving SQL problems.

---

## 📚 Table of Contents

1. [Database & Table Operations](#1-database--table-operations)
2. [Data Types](#2-data-types)
3. [INSERT — Adding Data](#3-insert--adding-data)
4. [SELECT — Querying Data](#4-select--querying-data)
5. [WHERE — Filtering Rows](#5-where--filtering-rows)
6. [ORDER BY — Sorting](#6-order-by--sorting)
7. [LIMIT & OFFSET — Pagination](#7-limit--offset--pagination)
8. [UPDATE — Modifying Data](#8-update--modifying-data)
9. [DELETE — Removing Data](#9-delete--removing-data)
10. [Aggregate Functions](#10-aggregate-functions)
11. [GROUP BY & HAVING](#11-group-by--having)
12. [JOINs](#12-joins)
13. [Subqueries](#13-subqueries)
14. [Set Operations](#14-set-operations)
15. [String Functions](#15-string-functions)
16. [Numeric Functions](#16-numeric-functions)
17. [Date & Time Functions](#17-date--time-functions)
18. [CASE — Conditional Logic](#18-case--conditional-logic)
19. [NULL Handling](#19-null-handling)
20. [Constraints](#20-constraints)
21. [Indexes](#21-indexes)
22. [Views](#22-views)
23. [Stored Procedures & Functions](#23-stored-procedures--functions)
24. [Transactions](#24-transactions)
25. [Window Functions](#25-window-functions)
26. [CTEs (Common Table Expressions)](#26-ctes-common-table-expressions)
27. [Advanced Techniques](#27-advanced-techniques)
28. [Performance Tips](#28-performance-tips)

---

## Sample Tables Used in This Guide

```sql
-- employees table
CREATE TABLE employees (
    id          INT PRIMARY KEY,
    name        VARCHAR(100),
    department  VARCHAR(50),
    salary      DECIMAL(10,2),
    manager_id  INT,
    hire_date   DATE
);

-- departments table
CREATE TABLE departments (
    id    INT PRIMARY KEY,
    name  VARCHAR(50),
    budget DECIMAL(15,2)
);

-- orders table
CREATE TABLE orders (
    id          INT PRIMARY KEY,
    customer_id INT,
    amount      DECIMAL(10,2),
    status      VARCHAR(20),
    order_date  DATE
);
```

---

## 1. Database & Table Operations

### Create / Drop Database
```sql
CREATE DATABASE company_db;
DROP DATABASE company_db;
USE company_db;                  -- MySQL
-- PostgreSQL uses \c company_db in psql
```

### Create Table
```sql
CREATE TABLE employees (
    id         INT PRIMARY KEY AUTO_INCREMENT,   -- MySQL
    -- id      SERIAL PRIMARY KEY,               -- PostgreSQL
    name       VARCHAR(100) NOT NULL,
    salary     DECIMAL(10, 2) DEFAULT 0.00,
    hire_date  DATE
);
```

### Alter Table
```sql
-- Add column
ALTER TABLE employees ADD COLUMN email VARCHAR(150);

-- Rename column
ALTER TABLE employees RENAME COLUMN email TO work_email;

-- Change column type
ALTER TABLE employees MODIFY COLUMN salary FLOAT;          -- MySQL
-- ALTER TABLE employees ALTER COLUMN salary TYPE FLOAT;   -- PostgreSQL

-- Drop column
ALTER TABLE employees DROP COLUMN work_email;

-- Rename table
RENAME TABLE employees TO staff;        -- MySQL
-- ALTER TABLE employees RENAME TO staff; -- PostgreSQL
```

### Drop / Truncate Table
```sql
DROP TABLE employees;           -- Removes table + data permanently
TRUNCATE TABLE employees;       -- Removes all rows, keeps structure (fast)
```

---

## 2. Data Types

| Category   | Type                              | Example                   |
|------------|-----------------------------------|---------------------------|
| Integer    | INT, BIGINT, SMALLINT, TINYINT    | `age INT`                 |
| Decimal    | DECIMAL(p,s), FLOAT, DOUBLE       | `salary DECIMAL(10,2)`    |
| String     | VARCHAR(n), CHAR(n), TEXT         | `name VARCHAR(100)`       |
| Boolean    | BOOLEAN / TINYINT(1)              | `is_active BOOLEAN`       |
| Date/Time  | DATE, TIME, DATETIME, TIMESTAMP   | `hire_date DATE`          |
| Binary     | BLOB, BINARY                      | `photo BLOB`              |
| JSON       | JSON                              | `meta JSON`               |

---

## 3. INSERT — Adding Data

```sql
-- Single row
INSERT INTO employees (name, department, salary, hire_date)
VALUES ('Alice', 'Engineering', 85000, '2021-03-15');

-- Multiple rows
INSERT INTO employees (name, department, salary, hire_date)
VALUES 
    ('Bob',   'Marketing',   60000, '2020-07-01'),
    ('Carol', 'Engineering', 92000, '2019-11-20'),
    ('Dave',  'HR',          55000, '2022-01-10');

-- Insert from another table
INSERT INTO archived_employees
SELECT * FROM employees WHERE hire_date < '2018-01-01';

-- Insert or ignore if duplicate (MySQL)
INSERT IGNORE INTO employees (id, name) VALUES (1, 'Alice');

-- Upsert (Insert or Update on conflict) — PostgreSQL
INSERT INTO employees (id, name, salary)
VALUES (1, 'Alice', 90000)
ON CONFLICT (id) DO UPDATE SET salary = EXCLUDED.salary;

-- Upsert — MySQL
INSERT INTO employees (id, name, salary)
VALUES (1, 'Alice', 90000)
ON DUPLICATE KEY UPDATE salary = VALUES(salary);
```

---

## 4. SELECT — Querying Data

```sql
-- All columns
SELECT * FROM employees;

-- Specific columns
SELECT name, department, salary FROM employees;

-- Column alias
SELECT name AS employee_name, salary AS monthly_pay
FROM employees;

-- Calculated column
SELECT name, salary, salary * 12 AS annual_salary FROM employees;

-- Distinct values
SELECT DISTINCT department FROM employees;

-- Distinct count
SELECT COUNT(DISTINCT department) AS dept_count FROM employees;

-- Literal / constant column
SELECT name, 'Active' AS status FROM employees;
```

---

## 5. WHERE — Filtering Rows

```sql
-- Comparison operators
SELECT * FROM employees WHERE salary > 70000;
SELECT * FROM employees WHERE department = 'Engineering';
SELECT * FROM employees WHERE salary != 60000;

-- Logical operators
SELECT * FROM employees WHERE salary > 70000 AND department = 'Engineering';
SELECT * FROM employees WHERE department = 'HR' OR department = 'Marketing';
SELECT * FROM employees WHERE NOT department = 'HR';

-- BETWEEN (inclusive)
SELECT * FROM employees WHERE salary BETWEEN 60000 AND 90000;

-- IN / NOT IN
SELECT * FROM employees WHERE department IN ('Engineering', 'Marketing');
SELECT * FROM employees WHERE id NOT IN (1, 3, 5);

-- LIKE — pattern matching
SELECT * FROM employees WHERE name LIKE 'A%';       -- Starts with A
SELECT * FROM employees WHERE name LIKE '%son';      -- Ends with son
SELECT * FROM employees WHERE name LIKE '%li%';      -- Contains li
SELECT * FROM employees WHERE name LIKE '_o%';       -- 2nd char is o

-- IS NULL / IS NOT NULL
SELECT * FROM employees WHERE manager_id IS NULL;
SELECT * FROM employees WHERE manager_id IS NOT NULL;
```

---

## 6. ORDER BY — Sorting

```sql
-- Ascending (default)
SELECT * FROM employees ORDER BY salary;
SELECT * FROM employees ORDER BY salary ASC;

-- Descending
SELECT * FROM employees ORDER BY salary DESC;

-- Multiple columns
SELECT * FROM employees ORDER BY department ASC, salary DESC;

-- By column position
SELECT name, salary FROM employees ORDER BY 2 DESC;  -- 2 = salary

-- NULLS LAST / FIRST (PostgreSQL)
SELECT * FROM employees ORDER BY manager_id NULLS LAST;
```

---

## 7. LIMIT & OFFSET — Pagination

```sql
-- First 5 rows
SELECT * FROM employees LIMIT 5;

-- Skip 10, get next 5 (page 3 if page size = 5)
SELECT * FROM employees LIMIT 5 OFFSET 10;

-- Top 3 highest salaries
SELECT name, salary FROM employees ORDER BY salary DESC LIMIT 3;

-- SQL Server / MS Access uses TOP
-- SELECT TOP 5 * FROM employees;

-- Oracle uses ROWNUM or FETCH
-- SELECT * FROM employees FETCH FIRST 5 ROWS ONLY;
```

---

## 8. UPDATE — Modifying Data

```sql
-- Update single column
UPDATE employees SET salary = 95000 WHERE id = 1;

-- Update multiple columns
UPDATE employees 
SET salary = 95000, department = 'Senior Engineering'
WHERE id = 1;

-- Update based on calculation
UPDATE employees SET salary = salary * 1.10 WHERE department = 'Engineering';

-- Update using subquery
UPDATE employees 
SET salary = salary * 1.05
WHERE department IN (SELECT name FROM departments WHERE budget > 100000);

-- Update all rows (⚠️ no WHERE — affects everyone)
UPDATE employees SET is_active = 1;
```

---

## 9. DELETE — Removing Data

```sql
-- Delete specific rows
DELETE FROM employees WHERE id = 5;

-- Delete with condition
DELETE FROM employees WHERE department = 'HR' AND salary < 40000;

-- Delete with subquery
DELETE FROM employees 
WHERE department NOT IN (SELECT name FROM departments);

-- Delete all rows (keeps table structure)
DELETE FROM employees;       -- Logged, can be rolled back
TRUNCATE TABLE employees;    -- Faster, usually can't roll back
```

---

## 10. Aggregate Functions

```sql
-- COUNT
SELECT COUNT(*) AS total_employees FROM employees;
SELECT COUNT(manager_id) AS has_manager FROM employees;  -- ignores NULLs

-- SUM
SELECT SUM(salary) AS total_payroll FROM employees;

-- AVG
SELECT AVG(salary) AS avg_salary FROM employees;

-- MIN / MAX
SELECT MIN(salary) AS lowest, MAX(salary) AS highest FROM employees;

-- Combine aggregates
SELECT 
    COUNT(*)       AS headcount,
    AVG(salary)    AS avg_salary,
    MIN(salary)    AS min_salary,
    MAX(salary)    AS max_salary,
    SUM(salary)    AS total_payroll
FROM employees;
```

---

## 11. GROUP BY & HAVING

```sql
-- Count employees per department
SELECT department, COUNT(*) AS headcount
FROM employees
GROUP BY department;

-- Average salary per department
SELECT department, AVG(salary) AS avg_salary
FROM employees
GROUP BY department;

-- HAVING — filter groups (like WHERE but for aggregates)
SELECT department, AVG(salary) AS avg_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 70000;

-- WHERE filters rows BEFORE grouping; HAVING filters AFTER
SELECT department, COUNT(*) AS headcount
FROM employees
WHERE hire_date >= '2020-01-01'         -- filter rows first
GROUP BY department
HAVING COUNT(*) > 2;                    -- then filter groups

-- GROUP BY multiple columns
SELECT department, YEAR(hire_date) AS year, COUNT(*) AS hires
FROM employees
GROUP BY department, YEAR(hire_date)
ORDER BY department, year;

-- WITH ROLLUP — adds subtotals and grand total
SELECT department, COUNT(*) AS cnt
FROM employees
GROUP BY department WITH ROLLUP;
```

---

## 12. JOINs

```sql
-- INNER JOIN — only matching rows in both tables
SELECT e.name, e.salary, d.name AS dept_name
FROM employees e
INNER JOIN departments d ON e.department = d.name;

-- LEFT JOIN — all rows from left + matching from right (NULL if no match)
SELECT e.name, d.budget
FROM employees e
LEFT JOIN departments d ON e.department = d.name;

-- RIGHT JOIN — all rows from right + matching from left
SELECT e.name, d.name AS dept_name
FROM employees e
RIGHT JOIN departments d ON e.department = d.name;

-- FULL OUTER JOIN — all rows from both sides
SELECT e.name, d.name AS dept_name
FROM employees e
FULL OUTER JOIN departments d ON e.department = d.name;
-- MySQL workaround:
-- SELECT ... FROM a LEFT JOIN b ON ... 
-- UNION 
-- SELECT ... FROM a RIGHT JOIN b ON ...

-- CROSS JOIN — every combination (cartesian product)
SELECT e.name, d.name
FROM employees e
CROSS JOIN departments d;

-- SELF JOIN — join a table to itself (e.g., find manager name)
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;

-- Join with multiple conditions
SELECT e.name, o.amount
FROM employees e
JOIN orders o ON e.id = o.customer_id AND o.status = 'completed';

-- Three-table join
SELECT e.name, d.name AS department, o.amount
FROM employees e
JOIN departments d ON e.department = d.name
JOIN orders o ON o.customer_id = e.id;
```

---

## 13. Subqueries

```sql
-- Scalar subquery (returns single value)
SELECT name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- Subquery in FROM (derived table)
SELECT dept, avg_sal
FROM (
    SELECT department AS dept, AVG(salary) AS avg_sal
    FROM employees
    GROUP BY department
) AS dept_avg
WHERE avg_sal > 65000;

-- Subquery in SELECT (correlated)
SELECT name,
       salary,
       (SELECT AVG(salary) FROM employees) AS company_avg
FROM employees;

-- EXISTS — check if subquery returns any rows
SELECT name FROM departments d
WHERE EXISTS (
    SELECT 1 FROM employees e WHERE e.department = d.name
);

-- NOT EXISTS
SELECT name FROM departments d
WHERE NOT EXISTS (
    SELECT 1 FROM employees e WHERE e.department = d.name
);

-- IN with subquery
SELECT * FROM employees
WHERE department IN (SELECT name FROM departments WHERE budget > 50000);

-- Correlated subquery (references outer query)
SELECT name, salary, department
FROM employees e1
WHERE salary = (
    SELECT MAX(salary) FROM employees e2
    WHERE e2.department = e1.department
);
```

---

## 14. Set Operations

```sql
-- UNION — combine results, remove duplicates
SELECT name FROM employees
UNION
SELECT name FROM archived_employees;

-- UNION ALL — combine results, keep duplicates (faster)
SELECT name FROM employees
UNION ALL
SELECT name FROM archived_employees;

-- INTERSECT — rows present in BOTH (PostgreSQL, SQL Server)
SELECT department FROM employees
INTERSECT
SELECT name FROM departments;

-- EXCEPT / MINUS — rows in first but not in second
SELECT name FROM departments
EXCEPT                                   -- PostgreSQL / SQL Server
SELECT DISTINCT department FROM employees;
-- Use MINUS in Oracle
```

---

## 15. String Functions

```sql
-- Length
SELECT name, LENGTH(name) AS name_len FROM employees;
-- SQL Server: LEN(name)

-- Upper / Lower
SELECT UPPER(name), LOWER(department) FROM employees;

-- Trim whitespace
SELECT TRIM('  hello  ');           -- 'hello'
SELECT LTRIM('  hello');            -- 'hello'
SELECT RTRIM('hello  ');            -- 'hello'

-- Substring
SELECT SUBSTRING(name, 1, 3) FROM employees;    -- first 3 chars
-- PostgreSQL: SUBSTR(name, 1, 3)

-- Concatenation
SELECT CONCAT(name, ' - ', department) AS label FROM employees;
-- PostgreSQL also supports: name || ' - ' || department

-- Replace
SELECT REPLACE(department, 'Engineering', 'Eng') FROM employees;

-- Position of substring
SELECT LOCATE('li', name) FROM employees;       -- MySQL
-- POSITION('li' IN name)                        -- Standard SQL

-- Left / Right
SELECT LEFT(name, 3), RIGHT(name, 3) FROM employees;

-- Repeat & Reverse
SELECT REPEAT('*', 5);          -- '*****'
SELECT REVERSE('hello');        -- 'olleh'

-- Format a number as string
SELECT FORMAT(salary, 2) FROM employees;        -- MySQL: '85,000.00'

-- String padding
SELECT LPAD('42', 5, '0');      -- '00042'
SELECT RPAD('hi', 5, '!');      -- 'hi!!!'
```

---

## 16. Numeric Functions

```sql
-- Rounding
SELECT ROUND(3.14159, 2);       -- 3.14
SELECT CEIL(4.1);               -- 5    (CEILING in some DBs)
SELECT FLOOR(4.9);              -- 4

-- Absolute value
SELECT ABS(-42);                -- 42

-- Modulo
SELECT 17 MOD 5;                -- 2
SELECT 17 % 5;                  -- 2 (MySQL, SQL Server)

-- Power & Square Root
SELECT POWER(2, 10);            -- 1024
SELECT SQRT(144);               -- 12

-- Truncate (no rounding)
SELECT TRUNCATE(3.999, 1);      -- 3.9 (MySQL)
-- TRUNC(3.999, 1)              -- PostgreSQL

-- Random number (0 to 1)
SELECT RAND();                  -- MySQL
-- SELECT RANDOM()              -- PostgreSQL

-- Greatest / Least
SELECT GREATEST(10, 20, 5);    -- 20
SELECT LEAST(10, 20, 5);       -- 5
```

---

## 17. Date & Time Functions

```sql
-- Current date/time
SELECT NOW();                   -- 2024-05-11 10:30:00
SELECT CURDATE();               -- 2024-05-11   (MySQL)
SELECT CURTIME();               -- 10:30:00     (MySQL)
-- PostgreSQL: CURRENT_DATE, CURRENT_TIME, NOW()

-- Extract parts
SELECT YEAR(hire_date),
       MONTH(hire_date),
       DAY(hire_date)
FROM employees;

-- PostgreSQL extract
SELECT EXTRACT(YEAR FROM hire_date) AS yr FROM employees;

-- Add / Subtract intervals
SELECT DATE_ADD(hire_date, INTERVAL 1 YEAR) FROM employees;     -- MySQL
SELECT DATE_SUB(hire_date, INTERVAL 30 DAY) FROM employees;     -- MySQL
-- PostgreSQL: hire_date + INTERVAL '1 year'

-- Difference between dates
SELECT DATEDIFF(NOW(), hire_date) AS days_employed FROM employees; -- MySQL
-- PostgreSQL: NOW()::DATE - hire_date

-- Format date
SELECT DATE_FORMAT(hire_date, '%d-%m-%Y') FROM employees;       -- MySQL
-- PostgreSQL: TO_CHAR(hire_date, 'DD-MM-YYYY')

-- Day of week, month name
SELECT DAYNAME(hire_date), MONTHNAME(hire_date) FROM employees;

-- Convert string to date
SELECT STR_TO_DATE('15-03-2021', '%d-%m-%Y');   -- MySQL
-- PostgreSQL: TO_DATE('15-03-2021', 'DD-MM-YYYY')
```

---

## 18. CASE — Conditional Logic

```sql
-- Simple CASE
SELECT name,
       CASE department
           WHEN 'Engineering' THEN 'Tech'
           WHEN 'Marketing'   THEN 'Biz'
           ELSE 'Other'
       END AS dept_group
FROM employees;

-- Searched CASE
SELECT name, salary,
       CASE
           WHEN salary >= 90000 THEN 'Senior'
           WHEN salary >= 70000 THEN 'Mid'
           WHEN salary >= 50000 THEN 'Junior'
           ELSE 'Entry'
       END AS level
FROM employees;

-- CASE in ORDER BY
SELECT * FROM employees
ORDER BY CASE department
             WHEN 'Engineering' THEN 1
             WHEN 'Marketing'   THEN 2
             ELSE 3
         END;

-- CASE in aggregate (conditional count)
SELECT 
    COUNT(CASE WHEN salary >= 80000 THEN 1 END) AS senior_count,
    COUNT(CASE WHEN salary <  80000 THEN 1 END) AS junior_count
FROM employees;

-- IIF shorthand (SQL Server)
-- SELECT IIF(salary > 70000, 'High', 'Low') FROM employees;
```

---

## 19. NULL Handling

```sql
-- COALESCE — returns first non-NULL value
SELECT name, COALESCE(manager_id, 0) AS mgr FROM employees;
SELECT COALESCE(NULL, NULL, 'fallback');    -- 'fallback'

-- IFNULL (MySQL) / ISNULL (SQL Server)
SELECT IFNULL(manager_id, -1) FROM employees;

-- NULLIF — returns NULL if both args are equal
SELECT NULLIF(salary, 0) FROM employees;    -- returns NULL when salary=0

-- Filtering NULLs
SELECT * FROM employees WHERE manager_id IS NULL;
SELECT * FROM employees WHERE manager_id IS NOT NULL;

-- NULL in arithmetic — any operation with NULL = NULL
SELECT NULL + 100;      -- NULL
SELECT NULL = NULL;     -- NULL (not TRUE!)

-- Safe NULL comparison
SELECT * FROM employees
WHERE manager_id <=> NULL;          -- MySQL spaceship operator
-- WHERE manager_id IS NOT DISTINCT FROM NULL;  -- Standard SQL
```

---

## 20. Constraints

```sql
-- PRIMARY KEY
id INT PRIMARY KEY
-- or: PRIMARY KEY (id)

-- NOT NULL
name VARCHAR(100) NOT NULL

-- UNIQUE
email VARCHAR(150) UNIQUE

-- DEFAULT
is_active BOOLEAN DEFAULT TRUE

-- CHECK
salary DECIMAL(10,2) CHECK (salary >= 0)

-- FOREIGN KEY
manager_id INT,
FOREIGN KEY (manager_id) REFERENCES employees(id)
    ON DELETE SET NULL
    ON UPDATE CASCADE

-- Named constraint
CONSTRAINT chk_salary CHECK (salary BETWEEN 0 AND 999999)

-- Add constraint to existing table
ALTER TABLE employees
ADD CONSTRAINT uq_email UNIQUE (email);

-- Drop constraint
ALTER TABLE employees DROP CONSTRAINT uq_email;
```

---

## 21. Indexes

```sql
-- Create index (speeds up lookups on column)
CREATE INDEX idx_department ON employees(department);

-- Unique index
CREATE UNIQUE INDEX idx_email ON employees(email);

-- Composite index (good for queries filtering both columns)
CREATE INDEX idx_dept_salary ON employees(department, salary);

-- Full-text index (for LIKE searches on large text)
CREATE FULLTEXT INDEX idx_name ON employees(name);   -- MySQL

-- Drop index
DROP INDEX idx_department ON employees;              -- MySQL
-- DROP INDEX idx_department;                        -- PostgreSQL

-- View indexes on a table
SHOW INDEX FROM employees;                           -- MySQL

-- ✅ When to add an index:
-- - Columns used in WHERE, JOIN ON, ORDER BY
-- - Columns with high cardinality (many unique values)

-- ❌ When NOT to index:
-- - Small tables
-- - Columns updated very frequently
-- - Columns with very low cardinality (e.g., boolean)
```

---

## 22. Views

```sql
-- Create view
CREATE VIEW high_earners AS
SELECT name, department, salary
FROM employees
WHERE salary > 80000;

-- Use view like a table
SELECT * FROM high_earners ORDER BY salary DESC;

-- Create or replace view
CREATE OR REPLACE VIEW high_earners AS
SELECT name, department, salary
FROM employees
WHERE salary > 90000;

-- Updatable view (simple views can be updated)
UPDATE high_earners SET salary = 95000 WHERE name = 'Alice';

-- Drop view
DROP VIEW high_earners;

-- View with JOIN
CREATE VIEW employee_details AS
SELECT e.name, e.salary, d.name AS dept_name, d.budget
FROM employees e
JOIN departments d ON e.department = d.name;
```

---

## 23. Stored Procedures & Functions

```sql
-- Stored Procedure (MySQL)
DELIMITER $$
CREATE PROCEDURE get_dept_employees(IN dept_name VARCHAR(50))
BEGIN
    SELECT * FROM employees WHERE department = dept_name;
END$$
DELIMITER ;

-- Call a procedure
CALL get_dept_employees('Engineering');

-- Procedure with OUT parameter
DELIMITER $$
CREATE PROCEDURE count_dept(IN dept VARCHAR(50), OUT emp_count INT)
BEGIN
    SELECT COUNT(*) INTO emp_count
    FROM employees WHERE department = dept;
END$$
DELIMITER ;

CALL count_dept('Engineering', @cnt);
SELECT @cnt;

-- Stored Function (returns a value)
DELIMITER $$
CREATE FUNCTION annual_salary(monthly DECIMAL(10,2))
RETURNS DECIMAL(10,2) DETERMINISTIC
BEGIN
    RETURN monthly * 12;
END$$
DELIMITER ;

SELECT name, annual_salary(salary) AS annual FROM employees;

-- Drop procedure/function
DROP PROCEDURE get_dept_employees;
DROP FUNCTION annual_salary;
```

---

## 24. Transactions

```sql
-- Basic transaction
START TRANSACTION;
    UPDATE employees SET salary = salary + 5000 WHERE id = 1;
    UPDATE employees SET salary = salary - 5000 WHERE id = 2;
COMMIT;                     -- Makes changes permanent

-- Rollback on error
START TRANSACTION;
    DELETE FROM employees WHERE id = 10;
ROLLBACK;                   -- Undoes all changes in transaction

-- Savepoints
START TRANSACTION;
    INSERT INTO employees (name, salary) VALUES ('Test', 50000);
    SAVEPOINT before_update;
    UPDATE employees SET salary = 99999 WHERE name = 'Test';
ROLLBACK TO before_update;  -- Undo only the UPDATE
COMMIT;                     -- Commit the INSERT

-- Transaction isolation levels
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;   -- Prevents dirty reads
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;     -- Strictest (slowest)

-- Levels (low → high isolation):
-- READ UNCOMMITTED → READ COMMITTED → REPEATABLE READ → SERIALIZABLE
```

---

## 25. Window Functions

> Window functions perform calculations across a set of rows **without collapsing them** like GROUP BY does.

```sql
-- Syntax: function() OVER (PARTITION BY ... ORDER BY ...)

-- ROW_NUMBER — unique sequential number per partition
SELECT name, department, salary,
       ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS rank_in_dept
FROM employees;

-- RANK — same rank for ties, gaps in sequence
SELECT name, salary,
       RANK() OVER (ORDER BY salary DESC) AS rnk
FROM employees;

-- DENSE_RANK — same rank for ties, NO gaps
SELECT name, salary,
       DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rnk
FROM employees;

-- NTILE — divide rows into N buckets
SELECT name, salary,
       NTILE(4) OVER (ORDER BY salary DESC) AS quartile
FROM employees;

-- LAG — value from previous row
SELECT name, salary, hire_date,
       LAG(salary, 1) OVER (ORDER BY hire_date) AS prev_salary
FROM employees;

-- LEAD — value from next row
SELECT name, salary, hire_date,
       LEAD(salary, 1) OVER (ORDER BY hire_date) AS next_salary
FROM employees;

-- Running total (SUM)
SELECT name, salary, hire_date,
       SUM(salary) OVER (ORDER BY hire_date) AS running_total
FROM employees;

-- Moving average (last 3 rows)
SELECT name, salary,
       AVG(salary) OVER (ORDER BY hire_date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS moving_avg
FROM employees;

-- FIRST_VALUE / LAST_VALUE
SELECT name, department, salary,
       FIRST_VALUE(salary) OVER (PARTITION BY department ORDER BY salary DESC) AS dept_highest
FROM employees;

-- Percent rank (0 to 1)
SELECT name, salary,
       PERCENT_RANK() OVER (ORDER BY salary) AS pct_rank
FROM employees;

-- Cumulative distribution
SELECT name, salary,
       CUME_DIST() OVER (ORDER BY salary) AS cum_dist
FROM employees;
```

---

## 26. CTEs (Common Table Expressions)

```sql
-- Basic CTE
WITH dept_avg AS (
    SELECT department, AVG(salary) AS avg_sal
    FROM employees
    GROUP BY department
)
SELECT e.name, e.salary, d.avg_sal
FROM employees e
JOIN dept_avg d ON e.department = d.department
WHERE e.salary > d.avg_sal;

-- Multiple CTEs
WITH 
high_salary AS (
    SELECT * FROM employees WHERE salary > 80000
),
eng_dept AS (
    SELECT * FROM departments WHERE name = 'Engineering'
)
SELECT h.name, h.salary
FROM high_salary h
JOIN eng_dept e ON h.department = e.name;

-- Recursive CTE — great for hierarchies (org charts, trees)
WITH RECURSIVE org_chart AS (
    -- Anchor: top-level employees (no manager)
    SELECT id, name, manager_id, 1 AS level
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive: employees under each manager
    SELECT e.id, e.name, e.manager_id, o.level + 1
    FROM employees e
    JOIN org_chart o ON e.manager_id = o.id
)
SELECT * FROM org_chart ORDER BY level, name;

-- CTE for pagination
WITH ranked AS (
    SELECT *, ROW_NUMBER() OVER (ORDER BY salary DESC) AS rn
    FROM employees
)
SELECT * FROM ranked WHERE rn BETWEEN 11 AND 20;   -- Page 2 (10 per page)
```

---

## 27. Advanced Techniques

### Pivot (rows to columns)
```sql
-- MySQL: use conditional aggregation
SELECT department,
    SUM(CASE WHEN YEAR(hire_date) = 2021 THEN 1 ELSE 0 END) AS hires_2021,
    SUM(CASE WHEN YEAR(hire_date) = 2022 THEN 1 ELSE 0 END) AS hires_2022,
    SUM(CASE WHEN YEAR(hire_date) = 2023 THEN 1 ELSE 0 END) AS hires_2023
FROM employees
GROUP BY department;
```

### Find Duplicates
```sql
-- Find duplicate names
SELECT name, COUNT(*) AS cnt
FROM employees
GROUP BY name
HAVING COUNT(*) > 1;

-- Delete duplicates, keep the one with lowest id
DELETE FROM employees
WHERE id NOT IN (
    SELECT MIN(id) FROM employees GROUP BY name
);
```

### Running Difference
```sql
SELECT hire_date, salary,
       salary - LAG(salary) OVER (ORDER BY hire_date) AS salary_diff
FROM employees;
```

### Top N per Group (most common interview problem)
```sql
-- Top 2 earners per department using ROW_NUMBER
WITH ranked AS (
    SELECT *,
           ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS rn
    FROM employees
)
SELECT * FROM ranked WHERE rn <= 2;
```

### Gaps and Islands (consecutive ranges)
```sql
-- Find gaps in sequential IDs
SELECT id + 1 AS gap_start
FROM employees e1
WHERE NOT EXISTS (
    SELECT 1 FROM employees e2 WHERE e2.id = e1.id + 1
)
AND id < (SELECT MAX(id) FROM employees);
```

### Median Calculation
```sql
-- MySQL workaround for median
SELECT AVG(salary) AS median
FROM (
    SELECT salary,
           ROW_NUMBER() OVER (ORDER BY salary) AS rn,
           COUNT(*) OVER () AS cnt
    FROM employees
) t
WHERE rn IN (FLOOR((cnt + 1) / 2), CEIL((cnt + 1) / 2));
```

### String Aggregation (list of values)
```sql
-- MySQL
SELECT department, GROUP_CONCAT(name ORDER BY name SEPARATOR ', ') AS members
FROM employees
GROUP BY department;

-- PostgreSQL
SELECT department, STRING_AGG(name, ', ' ORDER BY name) AS members
FROM employees
GROUP BY department;
```

### JSON Functions (MySQL 5.7+ / PostgreSQL)
```sql
-- Extract from JSON column
SELECT JSON_EXTRACT(meta, '$.city') AS city FROM employees;         -- MySQL
-- SELECT meta->>'city' FROM employees;                             -- PostgreSQL

-- Check if key exists
SELECT * FROM employees WHERE JSON_CONTAINS_PATH(meta, 'one', '$.skills');
```

---

## 28. Performance Tips

```sql
-- ✅ Use indexes on WHERE, JOIN, ORDER BY columns
CREATE INDEX idx_dept ON employees(department);

-- ✅ SELECT only needed columns (avoid SELECT *)
SELECT name, salary FROM employees WHERE department = 'Engineering';

-- ✅ Use EXISTS instead of IN for large subqueries
-- Slower:
SELECT * FROM departments WHERE id IN (SELECT department_id FROM employees);
-- Faster:
SELECT * FROM departments d WHERE EXISTS (
    SELECT 1 FROM employees e WHERE e.department_id = d.id
);

-- ✅ Filter early — push WHERE conditions as close to data as possible

-- ✅ Use EXPLAIN to analyze query plan
EXPLAIN SELECT * FROM employees WHERE department = 'Engineering';
EXPLAIN ANALYZE SELECT * FROM employees WHERE salary > 80000;  -- PostgreSQL

-- ✅ Avoid functions on indexed columns in WHERE (prevents index use)
-- Bad:
WHERE YEAR(hire_date) = 2022
-- Better:
WHERE hire_date BETWEEN '2022-01-01' AND '2022-12-31'

-- ✅ Use LIMIT when you only need a few rows
SELECT * FROM employees ORDER BY salary DESC LIMIT 10;

-- ✅ Batch large INSERTs instead of one row at a time
INSERT INTO employees VALUES (1,'a',50000),(2,'b',60000),(3,'c',70000);

-- ✅ Avoid SELECT DISTINCT when GROUP BY achieves the same result
-- Avoid N+1 queries: use JOINs instead of looping queries in application code

-- ✅ Normalize data to reduce redundancy; denormalize selectively for read-heavy tables

-- ✅ Use connection pooling in applications to reduce connection overhead
```

---

## 🔖 Quick Reference Card

| Task | SQL |
|------|-----|
| Count rows | `SELECT COUNT(*) FROM t;` |
| Unique values | `SELECT DISTINCT col FROM t;` |
| Top N rows | `SELECT * FROM t ORDER BY col DESC LIMIT N;` |
| Conditional value | `CASE WHEN ... THEN ... ELSE ... END` |
| Replace NULL | `COALESCE(col, default_val)` |
| Pattern match | `col LIKE 'A%'` |
| Range filter | `col BETWEEN x AND y` |
| Aggregate per group | `GROUP BY col HAVING agg_func > value` |
| Combine tables | `JOIN t2 ON t1.id = t2.fk` |
| Rank rows | `RANK() / DENSE_RANK() OVER (ORDER BY col)` |
| Running total | `SUM(col) OVER (ORDER BY col2)` |
| Previous row | `LAG(col) OVER (ORDER BY col2)` |
| Hierarchy | `WITH RECURSIVE cte AS (...)` |
| Top N per group | `ROW_NUMBER() OVER (PARTITION BY grp ORDER BY col)` |
| Remove duplicates | `GROUP BY ... HAVING COUNT(*) > 1` |
| Explain query | `EXPLAIN SELECT ...` |

---

*Made for GitHub — bookmark this file and reference it while solving SQL problems!*
