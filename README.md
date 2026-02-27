# 🐘 PostgreSQL SQL Tutorial — Zero to Hero

A comprehensive guide to SQL using PostgreSQL, covering everything from the basics to advanced topics like window functions, CTEs, stored procedures, and triggers.

---

## 📚 Table of Contents

1. [Introduction](#1-introduction)
2. [Databases: CRUD & Core Queries](#2-databases-crud--core-queries)
3. [Data Types & Constraints](#3-data-types--constraints)
4. [Clauses & Operators](#4-clauses--operators)
5. [Aggregate Functions](#5-aggregate-functions)
6. [String Functions](#6-string-functions)
7. [Altering Tables](#7-altering-tables)
8. [Relationships & Joins](#8-relationships--joins)
9. [Views & HAVING](#9-views--having)
10. [Stored Routines](#10-stored-routines)
11. [Window Functions](#11-window-functions)
12. [CTEs (Common Table Expressions)](#12-ctes-common-table-expressions)
13. [Triggers](#13-triggers)

---

## 1. Introduction

### What is a Database?
- An **organised collection of data**
- A method to manipulate and access data

### Database vs DBMS vs RDBMS

| Term | Description |
|------|-------------|
| **Database** | Raw storage of data |
| **DBMS** | Software that manages the database (e.g., PostgreSQL, MySQL, Oracle) |
| **RDBMS** | A DBMS that stores data in structured **tables** (rows & columns) and uses SQL |

### SQL vs PostgreSQL

- **SQL** (Structured Query Language) — the language used to communicate with databases
  ```sql
  SELECT * FROM person_db;
  ```
- **PostgreSQL** — an open-source RDBMS that uses SQL

### Other Popular Databases
MongoDB, Oracle, MySQL, SQLite, MaxDB, Firebird, Redis

---

## 2. Databases: CRUD & Core Queries

### Useful `psql` Commands

| Command | Description |
|---------|-------------|
| `\l` | List all databases |
| `\q` | Quit |
| `\c db_name` | Connect to a database |
| `\dt` | List all tables |
| `\d tb_name` | Describe a table |
| `\d+ tb_name` | List all columns (detailed) |
| `\! cls` | Clear screen |

### List Databases
```sql
SELECT datname FROM pg_database;
-- or
\l
```

### Create a Database
```sql
CREATE DATABASE db_name;
```

### Connect to a Database
```sql
\c db_name;
```

### Drop a Database
```sql
DROP DATABASE db_name;
```

---

### Tables

#### Create a Table
```sql
CREATE TABLE person (
  id   INT,
  name VARCHAR(100),
  city VARCHAR(100)
);
```

#### Check a Table
```sql
\d TABLE_NAME
```

---

### INSERT (Create)
```sql
INSERT INTO person(id, name, city)
VALUES (101, 'Raju', 'Delhi');
```

### SELECT (Read)
```sql
SELECT * FROM person;
SELECT name FROM person;
```

### UPDATE
```sql
UPDATE person
SET city = 'London'
WHERE id = 2;
```

### DELETE
```sql
DELETE FROM person
WHERE name = 'Raju';
```

---

### Sample Dataset: `employees`

```sql
CREATE TABLE employees (
  emp_id    SERIAL PRIMARY KEY,
  fname     VARCHAR(50) NOT NULL,
  lname     VARCHAR(50) NOT NULL,
  email     VARCHAR(100),
  dept      VARCHAR(50),
  salary    NUMERIC(10,2),
  hire_date DATE
);

INSERT INTO employees (emp_id, fname, lname, email, dept, salary, hire_date)
VALUES
  (1,  'Raj',    'Sharma', 'raj.sharma@example.com',    'IT',        50000.00, '2020-01-15'),
  (2,  'Priya',  'Singh',  'priya.singh@example.com',   'HR',        45000.00, '2019-03-22'),
  (3,  'Arjun',  'Verma',  'arjun.verma@example.com',   'IT',        55000.00, '2021-06-01'),
  (4,  'Suman',  'Patel',  'suman.patel@example.com',   'Finance',   60000.00, '2018-07-30'),
  (5,  'Kavita', 'Rao',    'kavita.rao@example.com',    'HR',        47000.00, '2020-11-10'),
  (6,  'Amit',   'Gupta',  'amit.gupta@example.com',    'Marketing', 52000.00, '2020-09-25'),
  (7,  'Neha',   'Desai',  'neha.desai@example.com',    'IT',        48000.00, '2019-05-18'),
  (8,  'Rahul',  'Kumar',  'rahul.kumar@example.com',   'IT',        53000.00, '2021-02-14'),
  (9,  'Anjali', 'Mehta',  'anjali.mehta@example.com',  'Finance',   61000.00, '2018-12-03'),
  (10, 'Vijay',  'Nair',   'vijay.nair@example.com',    'Marketing', 50000.00, '2020-04-19');
```

---

## 3. Data Types & Constraints

### Common Data Types

| Category | Types |
|----------|-------|
| Numeric  | `INT`, `DOUBLE`, `FLOAT`, `DECIMAL` |
| String   | `VARCHAR(n)` |
| Date     | `DATE` |
| Boolean  | `BOOLEAN` |

#### DECIMAL precision
```sql
DECIMAL(5,2)  -- Total 5 digits, 2 after decimal point
-- e.g., 155.38, 119.12, 28.15
```

---

### Constraints

#### PRIMARY KEY
```sql
CREATE TABLE person (
  id   INT PRIMARY KEY,
  name VARCHAR(100)
);
```
- Uniquely identifies each record
- Must be UNIQUE and NOT NULL
- Only ONE primary key per table

#### NOT NULL
```sql
CREATE TABLE person (
  name VARCHAR(100) NOT NULL
);
```

#### DEFAULT
```sql
CREATE TABLE person (
  city VARCHAR(100) DEFAULT 'Unknown'
);
```

#### SERIAL (Auto Increment)
```sql
CREATE TABLE person (
  id SERIAL PRIMARY KEY
);
```

#### UNIQUE
```sql
CREATE TABLE person (
  email VARCHAR(100) UNIQUE
);
```

#### CHECK Constraint
```sql
-- Named constraint example: phone must be at least 10 digits
ALTER TABLE employees
ADD CONSTRAINT chk_phone CHECK (LENGTH(phone) >= 10);
```

---

## 4. Clauses & Operators

### WHERE
```sql
SELECT * FROM employees WHERE salary > 65000;
```

### Relational Operators
`=`, `!=`, `>`, `<`, `>=`, `<=`

### Logical Operators

```sql
-- AND: both conditions must be true
SELECT * FROM employees WHERE salary = 25000 AND dept = 'IT';

-- OR: either condition must be true
SELECT * FROM employees WHERE salary = 65000 OR dept = 'IT';
```

### IN / NOT IN
```sql
SELECT * FROM employees WHERE dept IN ('IT', 'HR', 'Finance');
SELECT * FROM employees WHERE dept NOT IN ('Marketing');
```

### BETWEEN
```sql
SELECT * FROM employees WHERE salary BETWEEN 40000 AND 65000;
```

### DISTINCT
```sql
SELECT DISTINCT fname FROM employees;
```

### ORDER BY
```sql
SELECT * FROM employees ORDER BY fname;           -- ASC (default)
SELECT * FROM employees ORDER BY salary DESC;     -- DESC
```

### LIMIT
```sql
SELECT * FROM employees LIMIT 3;
```

### LIKE / ILIKE (Pattern Matching)
```sql
SELECT * FROM employees WHERE dept LIKE '%Acc%';

-- Pattern examples:
-- Starts with 'A'       : LIKE 'A%'
-- Ends with 'A'         : LIKE '%A'
-- Contains 'A'          : LIKE '%A%'
-- Second char is 'A'    : LIKE '_A%'
-- Case-insensitive      : ILIKE '%john%'
```

### NOT LIKE
```sql
SELECT * FROM employees WHERE fname NOT LIKE 'A%';
```

### IS NULL
```sql
SELECT * FROM employees WHERE fname IS NULL;
```

---

## 5. Aggregate Functions

### COUNT
```sql
SELECT COUNT(*) FROM employees;
```

### MAX / MIN
```sql
SELECT MAX(salary) FROM employees;
SELECT MIN(salary) FROM employees;

-- Employee with max salary
SELECT emp_id, fname, salary FROM employees
WHERE salary = (SELECT MAX(salary) FROM employees);
```

### SUM / AVG
```sql
SELECT SUM(salary) FROM employees;
SELECT AVG(salary) FROM employees;
```

### GROUP BY
```sql
-- Unique departments
SELECT dept FROM employees GROUP BY dept;

-- Employee count per department
SELECT dept, COUNT(fname) FROM employees GROUP BY dept;
```

---

## 6. String Functions

### CONCAT
```sql
SELECT CONCAT(fname, ' ', lname) FROM employees;
```

### CONCAT_WS (with separator)
```sql
SELECT CONCAT_WS('-', fname, lname) FROM employees;
-- Result: Raj-Sharma
```

### SUBSTRING
```sql
SELECT SUBSTRING('Hey Buddy', 1, 4);
-- Result: Hey
```

### REPLACE
```sql
SELECT REPLACE('Hey Buddy', 'Hey', 'Hello');
-- Result: Hello Buddy
```

### REVERSE
```sql
SELECT REVERSE('Hello World');
```

### LENGTH
```sql
SELECT LENGTH('Hello World');
-- Result: 11
```

### UPPER / LOWER
```sql
SELECT UPPER('Hello World');  -- HELLO WORLD
SELECT LOWER('Hello World');  -- hello world
```

### Other Functions
```sql
SELECT LEFT('Abcdefghij', 3);         -- Abc
SELECT RIGHT('Abcdefghij', 4);        -- ghij
SELECT TRIM(' Alright! ');            -- Alright!
SELECT POSITION('OM' IN 'Thomas');    -- 3
```

---

## 7. Altering Tables

### Add a Column
```sql
ALTER TABLE employees ADD COLUMN phone VARCHAR(15);
```

### Drop a Column
```sql
ALTER TABLE employees DROP COLUMN phone;
```

### Rename a Column
```sql
ALTER TABLE employees RENAME COLUMN fname TO first_name;
```

### Rename a Table
```sql
ALTER TABLE employees RENAME TO staff;
```

### Modify Column Type
```sql
ALTER TABLE employees ALTER COLUMN salary TYPE FLOAT;
```

### Add DEFAULT to a Column
```sql
ALTER TABLE employees ALTER COLUMN dept SET DEFAULT 'General';
```

### Set NOT NULL
```sql
ALTER TABLE employees ALTER COLUMN email SET NOT NULL;
```

### Add / Drop Constraints
```sql
-- Add a named CHECK constraint
ALTER TABLE employees
ADD CONSTRAINT chk_salary CHECK (salary >= 0);

-- Drop a constraint
ALTER TABLE employees DROP CONSTRAINT chk_salary;
```

### CASE Expression
```sql
SELECT fname, salary,
  CASE
    WHEN salary > 55000 THEN 'High'
    WHEN salary BETWEEN 45000 AND 55000 THEN 'Medium'
    ELSE 'Low'
  END AS salary_band
FROM employees;
```

---

## 8. Relationships & Joins

### Types of Relationships

| Type | Example |
|------|---------|
| One to One (1:1) | Employee ↔ Employee Details |
| One to Many (1:N) | Employee → Tasks |
| Many to Many (M:N) | Books ↔ Authors |

---

### Foreign Keys

```sql
CREATE TABLE customers (
  cust_id   SERIAL PRIMARY KEY,
  cust_name VARCHAR(100) NOT NULL
);

CREATE TABLE orders (
  ord_id   SERIAL PRIMARY KEY,
  ord_date DATE NOT NULL,
  price    NUMERIC NOT NULL,
  cust_id  INTEGER NOT NULL,
  FOREIGN KEY (cust_id) REFERENCES customers(cust_id)
);
```

### Sample Data
```sql
INSERT INTO customers (cust_name)
VALUES ('Raju'), ('Sham'), ('Paul'), ('Alex');

INSERT INTO orders (ord_date, cust_id, price)
VALUES
  ('2024-01-01', 1, 250.00),
  ('2024-01-15', 1, 300.00),
  ('2024-02-01', 2, 150.00),
  ('2024-03-01', 3, 450.00),
  ('2024-04-04', 2, 550.00);
```

---

### Joins

#### CROSS JOIN
Every row from table A combined with every row from table B.
```sql
SELECT * FROM customers CROSS JOIN orders;
```

#### INNER JOIN
Returns rows with a match in **both** tables.
```sql
SELECT customers.cust_name, orders.price
FROM customers
INNER JOIN orders ON customers.cust_id = orders.cust_id;
```

#### Inner Join with GROUP BY
```sql
SELECT customers.cust_name, COUNT(orders.ord_id) AS total_orders
FROM customers
INNER JOIN orders ON customers.cust_id = orders.cust_id
GROUP BY customers.cust_name;
```

#### LEFT JOIN
Returns **all rows from the left** table + matched rows from the right.
```sql
SELECT customers.cust_name, orders.price
FROM customers
LEFT JOIN orders ON customers.cust_id = orders.cust_id;
```

#### RIGHT JOIN
Returns **all rows from the right** table + matched rows from the left.
```sql
SELECT customers.cust_name, orders.price
FROM customers
RIGHT JOIN orders ON customers.cust_id = orders.cust_id;
```

---

### Many-to-Many Relationship

```sql
CREATE TABLE students (
  id           SERIAL PRIMARY KEY,
  student_name VARCHAR(100)
);

CREATE TABLE courses (
  id          SERIAL PRIMARY KEY,
  course_name VARCHAR(100),
  fees        NUMERIC
);

-- Junction / Bridge table
CREATE TABLE enrollment (
  student_id INT REFERENCES students(id),
  course_id  INT REFERENCES courses(id),
  PRIMARY KEY (student_id, course_id)
);
```

### CASCADE ON DELETE
```sql
CREATE TABLE orders (
  ord_id  SERIAL PRIMARY KEY,
  cust_id INTEGER REFERENCES customers(cust_id) ON DELETE CASCADE
);
-- Deleting a customer will automatically delete their orders
```

---

## 9. Views & HAVING

### Views
```sql
CREATE VIEW it_employees AS
SELECT * FROM employees WHERE dept = 'IT';

-- Query the view
SELECT * FROM it_employees;
```

### HAVING Clause
Use `HAVING` to filter **grouped** results (like `WHERE` but for aggregates).
```sql
SELECT dept, AVG(salary) AS avg_salary
FROM employees
GROUP BY dept
HAVING AVG(salary) > 50000;
```

### GROUP BY ROLLUP
```sql
SELECT dept, SUM(salary)
FROM employees
GROUP BY ROLLUP(dept);
```

---

## 10. Stored Routines

A **Stored Routine** is an SQL statement or set of statements stored on the database server and reusable multiple times.

### Types
- **Stored Procedure** — performs operations (INSERT, UPDATE, DELETE, SELECT)
- **User Defined Function (UDF)** — performs operations and **returns a value**

---

### Stored Procedures

#### Update Employee Salary
```sql
CREATE OR REPLACE PROCEDURE update_emp_salary(
  p_employee_id INT,
  p_new_salary  NUMERIC
)
LANGUAGE plpgsql
AS $$
BEGIN
  UPDATE employees
  SET salary = p_new_salary
  WHERE emp_id = p_employee_id;
END;
$$;

-- Call the procedure
CALL update_emp_salary(1, 60000);
```

#### Add a New Employee
```sql
CREATE OR REPLACE PROCEDURE add_employee(
  p_fname   VARCHAR,
  p_lname   VARCHAR,
  p_email   VARCHAR,
  p_dept    VARCHAR,
  p_salary  NUMERIC
)
LANGUAGE plpgsql
AS $$
BEGIN
  INSERT INTO employees (fname, lname, email, dept, salary)
  VALUES (p_fname, p_lname, p_email, p_dept, p_salary);
END;
$$;

-- Call the procedure
CALL add_employee('John', 'Doe', 'john@example.com', 'IT', 55000);
```

---

### User Defined Functions

#### Find Highest-Paid Employee per Department
```sql
CREATE OR REPLACE FUNCTION dept_max_sal_emp(dept_name VARCHAR)
RETURNS TABLE(emp_id INT, fname VARCHAR, salary NUMERIC)
AS $$
BEGIN
  RETURN QUERY
  SELECT e.emp_id, e.fname, e.salary
  FROM employees e
  WHERE e.dept = dept_name
    AND e.salary = (
      SELECT MAX(emp.salary)
      FROM employees emp
      WHERE emp.dept = dept_name
    );
END;
$$ LANGUAGE plpgsql;

-- Usage
SELECT * FROM dept_max_sal_emp('IT');
```

---

## 11. Window Functions

**Window functions** (analytic functions) perform calculations **across a set of rows related to the current row** without collapsing them, using an `OVER()` clause.

### Benefits
- Advanced analytics: running totals, moving averages, rankings, cumulative distributions
- **Non-aggregating**: individual row details are preserved
- Flexible: usable in `SELECT`, `ORDER BY`, and `HAVING`

### Common Window Functions

| Function | Description |
|----------|-------------|
| `ROW_NUMBER()` | Sequential row number within partition |
| `RANK()` | Rank with gaps for ties |
| `DENSE_RANK()` | Rank without gaps for ties |
| `LAG()` | Value from a previous row |
| `LEAD()` | Value from a subsequent row |

```sql
-- Rank employees by salary within each department
SELECT fname, dept, salary,
  RANK() OVER (PARTITION BY dept ORDER BY salary DESC) AS salary_rank
FROM employees;

-- Running total of salary
SELECT fname, salary,
  SUM(salary) OVER (ORDER BY emp_id) AS running_total
FROM employees;

-- Compare with previous row
SELECT fname, salary,
  LAG(salary) OVER (ORDER BY emp_id) AS prev_salary
FROM employees;
```

---

## 12. CTEs (Common Table Expressions)

A **CTE** is a temporary, named result set defined within a query to simplify complex SQL. It only exists for the duration of that single query.

```sql
WITH cte_name AS (
  -- your subquery here
)
SELECT * FROM cte_name;
```

### Use Case 1: Employees Above Department Average Salary
```sql
WITH AvgSal AS (
  SELECT dept, AVG(salary) AS avg_salary
  FROM employees
  GROUP BY dept
)
SELECT e.emp_id, e.fname, e.dept, e.salary, a.avg_salary
FROM employees e
JOIN AvgSal a ON e.dept = a.dept
WHERE e.salary > a.avg_salary;
```

### Use Case 2: Highest-Paid Employee Per Department
```sql
WITH HighestPaid AS (
  SELECT dept, MAX(salary) AS max_salary
  FROM employees
  GROUP BY dept
)
SELECT e.emp_id, e.fname, e.lname, e.dept, e.salary
FROM employees e
JOIN HighestPaid h ON e.dept = h.dept AND e.salary = h.max_salary;
```

> **Note:** A CTE can only be used **once** within the query it is defined in. It is not persisted.

---

## 13. Triggers

**Triggers** are special procedures that **automatically execute** in response to specific events (INSERT, UPDATE, DELETE) on a table.

### Use Case: Prevent Negative Salary
If a negative salary is inserted or updated, automatically set it to `0`.

```sql
-- Step 1: Create the trigger function
CREATE OR REPLACE FUNCTION prevent_negative_salary()
RETURNS TRIGGER AS $$
BEGIN
  IF NEW.salary < 0 THEN
    NEW.salary := 0;
  END IF;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Step 2: Attach the trigger to the table
CREATE TRIGGER trg_salary_check
BEFORE INSERT OR UPDATE ON employees
FOR EACH ROW
EXECUTE FUNCTION prevent_negative_salary();
```

---

## 🏋️ Exercises

### Section 4 — Clauses
1. Find all distinct departments in the database
2. Display records ordered by salary (high to low)
3. Show only the top 3 records from the table
4. Show records where first name starts with `'A'`
5. Show records where the length of `lname` is 4 characters

### Section 5 — Aggregates
1. Find total number of employees in the database
2. Find number of employees in each department
3. Find the lowest salary
4. Find the highest salary
5. Find total salary paid in a specific department
6. Find average salary per department

### Section 8 — Relationships Task
Create a shopping store schema with four tables — `customers`, `orders`, `products`, and `order_items` — demonstrating both one-to-many and many-to-many relationships.

```
customers          orders             products           order_items
---------          ------             --------           -----------
cust_id (PK)       ord_id (PK)        p_id (PK)          items_id (PK)
cust_name          ord_date           p_name             ord_id (FK)
                   cust_id (FK)       price              p_id (FK)
                                                         quantity
```

---

## 📌 Quick Reference Card

```sql
-- DDL
CREATE TABLE t (id SERIAL PRIMARY KEY, name VARCHAR(100));
ALTER TABLE t ADD COLUMN age INT;
DROP TABLE t;

-- DML
INSERT INTO t (name) VALUES ('Alice');
SELECT * FROM t WHERE id = 1;
UPDATE t SET name = 'Bob' WHERE id = 1;
DELETE FROM t WHERE id = 1;

-- Filtering
WHERE salary BETWEEN 40000 AND 60000
WHERE dept IN ('IT', 'HR')
WHERE fname LIKE 'A%'
WHERE email IS NULL

-- Aggregates
SELECT dept, COUNT(*), AVG(salary), MAX(salary), MIN(salary), SUM(salary)
FROM employees GROUP BY dept HAVING AVG(salary) > 50000;

-- Joins
FROM a INNER JOIN b ON a.id = b.a_id
FROM a LEFT  JOIN b ON a.id = b.a_id
FROM a RIGHT JOIN b ON a.id = b.a_id
FROM a CROSS JOIN b

-- CTE
WITH cte AS (SELECT ...) SELECT * FROM cte;

-- Window
RANK() OVER (PARTITION BY dept ORDER BY salary DESC)
```
