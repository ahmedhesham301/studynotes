# Creating Tables

## Syntax

```sql
CREATE TABLE table_name (
    column_name data_type [constraints],
    column_name data_type [constraints],
    ...
);
```

> Use **`IF NOT EXISTS`** to avoid errors if the table already exists.

## Data Types

| Data Type         | Description                                |
| ----------------- | ------------------------------------------ |
| `SERIAL`          | Auto-increment integer (primary keys)      |
| `INTEGER` / `INT` | Whole numbers                              |
| `VARCHAR(n)`      | Variable-length string with max length `n` |
| `TEXT`            | Unlimited-length string                    |
| `DATE`            | Calendar date (YYYY-MM-DD)                 |
| `TIMESTAMP`       | Date and time                              |
| `BOOLEAN`         | `TRUE` or `FALSE`                          |
| `NUMERIC(p,s)`    | Fixed-point numbers with precision/scale   |

## Constraints

| Constraint      | Description                                                                  |
| --------------- | ---------------------------------------------------------------------------- |
| `PRIMARY KEY`   | Unique identifier for each row                                               |
| `NOT NULL`      | Column must have a value                                                     |
| `UNIQUE`        | No duplicate values allowed                                                  |
| `CHECK (expr)`  | Must satisfy a condition                                                     |
| `DEFAULT value` | Assigns a default value if none is provided                                  |
| `REFERENCES`    | Foreign key constraint linking to another table                              |
| `ON DELETE`     | Defines behavior when the referenced row is deleted (used with `REFERENCES`) |

### `ON DELETE` Options

| Option                  | Behavior                                                                                          |
| ----------------------- | ------------------------------------------------------------------------------------------------- |
| `ON DELETE CASCADE`     | Deletes child rows automatically when the parent row is deleted                                   |
| `ON DELETE SET NULL`    | Sets the foreign key column in child rows to `NULL` when the parent row is deleted                |
| `ON DELETE RESTRICT`    | Prevents deleting the parent row if there are any child rows referencing it (default in many DBs) |
| `ON DELETE NO ACTION`   | Similar to `RESTRICT`, but enforcement is deferred until the end of the transaction               |
| `ON DELETE SET DEFAULT` | Sets the foreign key column to its default value when the parent row is deleted                   |

# Inserting

## Syntax

```sql
INSERT INTO table_name (column1, column2, ...)
VALUES (value1, value2, ...);
```

## Insert into All Columns **Not Recommended**

```sql
INSERT INTO employees
VALUES (1, 'John', 'Doe', 'john@example.com', '2025-08-09', 50000.00, 2);
```

> **Order matters** — values must match the table’s column order.

## Insert into Specific Columns

```sql
INSERT INTO employees (first_name, last_name, salary)
VALUES ('Jane', 'Smith', 60000.00);
```

>Columns not listed will use **DEFAULT** values (if defined) or `NULL`.

## Insert Multiple Rows

```sql
INSERT INTO employees (first_name, last_name, salary)
VALUES
    ('Alice', 'Brown', 55000.00),
    ('Bob', 'White', 47000.00),
    ('Charlie', 'Green', 62000.00);
```

# Querying Data

## Basics

### Basic SELECT Syntax

```sql
SELECT column1, column2, ...
FROM table_name;
```

### Selecting All Columns

```sql
SELECT * FROM employees;
```

### Filtering Rows (`WHERE`)

```sql
SELECT * FROM employees
WHERE salary > 50000;
```

- **Comparison operators**: `=`, `!=`, `<`, `<=`, `>`, `>=` ,`IN`,`NOT IN`,`BETWEEN`
- **Logical operators**: `AND`, `OR`, `NOT`
- **Pattern matching**:
  - `LIKE 'A%'` → starts with "A"
  - `LIKE '%son'` → ends with "son"
  - `ILIKE` → case-insensitive pattern match

### Selecting distinct records

```sql
SELECT DISTINCT department
FROM employees;
```

```sql
SELECT COUNT(DISTINCT department)
FROM employees;
```

### Aliases (AS)

```sql
SELECT first_name AS fname, last_name AS lname
FROM employees AS e;
```

### Conditional Expressions (`CASE`)

```sql
SELECT first_name,
       department,
       CASE department
           WHEN 'HR' THEN 'Human Resources'
           WHEN 'IT' THEN 'Tech Department'
           ELSE 'Other'
       END AS department_name
FROM employees;
```

### Aggregation Functions

| Function   | Description    |
| ---------- | -------------- |
| `COUNT(*)` | Number of rows |
| `SUM()`    | Sum of values  |
| `AVG()`    | Average value  |
| `MIN()`    | Minimum value  |
| `MAX()`    | Maximum value  |

### String Operations & Functions

| OP/Function  | Description                                                       |
| ------------ | ----------------------------------------------------------------- |
| \|\|         | Join two strings                                                  |
| `CONCAT()`   | Join two strings                                                  |
| `LOWER()`    | Gives a lower case string                                         |
| `UPPER()`    | Giver a upper case string                                         |
| `LENGTH()`   | Gives the number of chars in a string                             |
| `GREATEST()` | Returns the **largest** value from the given list of expressions. |
| `LEAST()`    | Returns the **smallest** value from the given list.               |

## Sorting Results (`ORDER BY`)

### basic syntax

```sql
SELECT first_name, salary
FROM employees
ORDER BY salary DESC;
```

- `ASC` → ascending (default)
- `DESC` → descending

### Multiple Columns

Sorts by the first column, then by the next in case of ties.

```sql
SELECT * FROM employees
ORDER BY department ASC, salary DESC;
```

## Joining Tables

### Basic syntax

```sql
SELECT column_list
FROM table1
[JOIN_TYPE] JOIN table2
ON table1.column_name = table2.column_name;
```

### Types of Joins

#### INNER JOIN

- Returns rows that have matching values in both tables.
- Most common join type.
- Default if no join type specified

```sql
SELECT *
FROM orders
INNER JOIN customers
ON orders.customer_id = customers.id;
```

> `WHERE` statement must come after the join
>
#### LEFT/RIGHT JOIN (or LEFT/RIGHT OUTER JOIN)

- Returns **all rows** from the left/right table, and matching rows from the right/left table.
- Non-matching rows from the right table appear as `NULL`.

```sql
SELECT *
FROM orders
LEFT JOIN customers
ON orders.customer_id = customers.id;
```

#### FULL JOIN (or FULL OUTER JOIN)

- Returns **all rows** from both tables, matching where possible.
- Non-matching rows get `NULL` in columns from the other table.

```sql
SELECT *
FROM orders
FULL JOIN customers
ON orders.customer_id = customers.id;
```

## `GROUP BY`

The **`GROUP BY`** clause in PostgreSQL is used to **arrange identical data into groups**. It is often combined with **aggregate functions**.

```sql
SELECT column1, aggregate_function(column2)
FROM table_name
WHERE condition
GROUP BY column1;
```

## `HAVING`

- The `HAVING` clause is used to **filter groups** after an aggregation has been applied.
- It works **with `GROUP BY`** and aggregate functions like `COUNT()`, `SUM()`, `AVG()`, `MIN()`, `MAX()`.
- Unlike `WHERE`, which filters **rows before grouping**, `HAVING` filters **groups after they have been formed**.

```sql
SELECT column_name, aggregate_function(column_name)
FROM table_name
GROUP BY column_name
HAVING condition;
```

## `OFFSET` and `LIMIT`
>
>Order Matters:

- Always use `ORDER BY` with `LIMIT`/`OFFSET` for consistent and predictable results.

>Performance Notes

- `OFFSET` still processes skipped rows internally, so large offsets can be slow.

### `OFFSET`

Skips a specified number of rows **before** starting to return rows.

```sql
SELECT * FROM employees OFFSET 10;
```

### `LIMIT`

Restricts the number of rows returned.

```sql
SELECT * FROM employees LIMIT 5;
```

## Set Operations

All require:

- The same number of columns.
- Compatible data types across corresponding columns.
- Column names in the result are taken from the **first query**.

### `UNION`

- **Combines** result sets of two or more queries.
- Removes duplicates by default.
- Use `UNION ALL` to keep duplicates.

```sql
SELECT column1, column2 FROM table1
UNION
SELECT column1, column2 FROM table2;
```

### `INTERSECT`

- Returns only the **common rows** between two queries.
- Removes duplicates by default.
- Use `INTERSECT ALL` to keep duplicates.

```sql
SELECT column1, column2 FROM table1
INTERSECT
SELECT column1, column2 FROM table2;
```

### `EXCEPT`

- Returns rows from the **first query** that are **not present** in the second query.
- Removes duplicates by default.
- Use `EXCEPT ALL` to keep duplicates.

```sql
SELECT column1, column2 FROM table1
EXCEPT
SELECT column1, column2 FROM table2;
```
## Common Table Expressions (CTEs) — `WITH ... AS ()`

A **CTE (Common Table Expression)** lets you define **temporary result sets** that exist only for the duration of a query.  
They make complex SQL queries easier to read and maintain.

```sql
WITH tags AS(
	SELECT user_id, created_at FROM caption_tags
    UNION ALL
    SELECT user_id, created_at FROM photo_tags
)

SELECT username, tags.created_at
FROM users
JOIN tags ON tags.user_id = users.id
WHERE tags.created_at < '2010-01-07';
```
## `ALL`,`ANY` ,`SOME`

### `ALL`

The expression must be greater than *every* value in the list or subquery.

```sql
SELECT *
FROM products
WHERE price > ALL (SELECT price FROM discounted_products);
```

### `ANY`, `SOME`

The expression must be true for *at least one* value in the list or subquery.

```sql
SELECT *
FROM orders
WHERE quantity > ANY (SELECT quantity FROM bulk_orders);
```

# Views (Fake tables)
- A **view** is a **virtual table** based on the result of a **SQL query**.
- It **does not store data physically**.

## Creating a view
```sql
CREATE VIEW view_name AS(
	SELECT column1, column2
	FROM table_name
	WHERE condition;
)

```
Example:
```sql
CREATE VIEW tags AS(
	SELECT id,created_at,user_id,post_id, 'photo_tag' AS type FROM photo_tags
	UNION ALL
	SELECT id,created_at,user_id,post_id, 'caption_tag' AS type FROM caption_tags
)
```

## Updating a view
```sql
CREATE OR REPLACE VIEW view_name AS(
	SELECT * FROM my_table
	WITH CHECK OPTION;
)
```
Example:
```sql
CREATE OR REPLACE VIEW recent_posts AS (
  SELECT *
  FROM posts
  ORDER BY created_at DESC
  LIMIT 15
);
```

## Dropping a view
```sql
DROP VIEW view_name;
```

# Materialized view
- A **Materialized View** is a **cached, physical copy of the result** of a query.  
- Unlike a regular view (which is just a stored SQL query), a materialized view stores data on disk and can be refreshed later.
## Creating a materialized view
```sql
CREATE MATERIALIZED VIEW view_name AS(
	SELECT ...
	FROM ...
)
WITH DATA;
```

## Refreshing a Materialized View
```sql
REFRESH MATERIALIZED VIEW view_name;
```


# Altering tables

is used to modify the structure of an existing table without dropping it.

```sql
ALTER TABLE table_name
action [, ...];
```

## Common Actions

| Action                 | Syntax Example                                                                                  | Notes                                                              |
| ---------------------- | ----------------------------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| **Rename Table**       | `ALTER TABLE old_name RENAME TO new_name;`                                                      | Changes the table name.                                            |
| **Add Column**         | `ALTER TABLE users ADD COLUMN age INT;`                                                         | Adds a new column. Fast unless `NOT NULL` without default is used. |
| **Drop Column**        | `ALTER TABLE users DROP COLUMN age CASCADE;`                                                    | Removes a column. `CASCADE` drops dependencies.                    |
| **Rename Column**      | `ALTER TABLE users RENAME COLUMN age TO user_age;`                                              | Renames an existing column.                                        |
| **Change Column Type** | `ALTER TABLE users ALTER COLUMN age TYPE BIGINT USING age::BIGINT;`                             | Converts values with `USING` if needed.                            |
| **Set Default**        | `ALTER TABLE users ALTER COLUMN age SET DEFAULT 18;`                                            | Assigns a default value for new rows.                              |
| **Drop Default**       | `ALTER TABLE users ALTER COLUMN age DROP DEFAULT;`                                              | Removes default value.                                             |
| **Set NOT NULL**       | `ALTER TABLE users ALTER COLUMN age SET NOT NULL;`                                              | Requires all rows to have a value.                                 |
| **Drop NOT NULL**      | `ALTER TABLE users ALTER COLUMN age DROP NOT NULL;`                                             | Allows `NULL` values.                                              |
| **Add Constraint**     | `ALTER TABLE orders ADD CONSTRAINT fk_customer FOREIGN KEY (cust_id) REFERENCES customers(id);` | Adds PK, FK, UNIQUE, CHECK, etc.                                   |
| **Drop Constraint**    | `ALTER TABLE orders DROP CONSTRAINT fk_customer CASCADE;`                                       | Removes a named constraint.                                        |
| **Rename Constraint**  | `ALTER TABLE orders RENAME CONSTRAINT fk_customer TO fk_client;`                                | Renames an existing constraint.                                    |
| **Change Owner**       | `ALTER TABLE users OWNER TO new_owner;`                                                         | Transfers ownership.                                               |
| **Move to Schema**     | `ALTER TABLE users SET SCHEMA archive;`                                                         | Moves table to another schema.                                     |

# Modify existing rows `UPDATE`

## update all rows

```sql
UPDATE employees
SET salary = salary * 1.05;
```
	
## Update with condition

```sql
UPDATE employees
SET salary = salary + 1000,
	status = 'active'
WHERE department = 'IT';
```

## Update using another table (`FROM`)

```sql
UPDATE employees e
SET department = d.dept_name
FROM departments d
WHERE e.dept_id = d.id;
```

# Transactions
A **transaction** is a sequence of one or more SQL statements executed as a single unit of work.
- The **main goal** is to ensure **data integrity and consistency**, even in the event of errors or system failures.
- A transaction **must be completed entirely** or **not executed at all** — this is known as the **“all-or-nothing”** property.

## Start a transaction
```postgresql
BEGIN;
```
## **Commit a transaction** (save all changes)
```postgresql
COMMIT;
```
## **Rollback a transaction** (undo all changes)
```postgresql
ROLLBACK;
```
# PostgreSQL Internal
## Commands
### see where PostgreSQL stores its **database files**.
 ```sql
 SHOW data_directory
 ```

### Lists all databases in the current PostgreSQL cluster with their **object IDs (OIDs)**.
```sql
SELECT oid, datname
FROM pg_database;
```

### list metadata about all relations
```sql
SELECT * FROM pg_class;
```

### get table,index,TOAST Size
```sql
SELECT pg_size_pretty(pg_relation_size('name'));
```

### lists **all indexes** (including auto generated ones)
```sql
SELECT relname,relkind
FROM pg_class
WHERE relkind = 'i';
```

### View Table Statistics
```sql
SELECT *
FROM pg_stats
WHERE tablename = 'users';
```
## concepts
### Storage Concepts

#### Heap File
- The **heap file** is how PostgreSQL physically stores **table data** on disk.
- Each table has its own heap file.
- Stored as a collection of **(blocks)**.
- A heap file is divided into many **blocks**.

#### Block (Page)
- A **block** (also called a **page**) is the smallest unit of storage I/O in PostgreSQL.
- Default size: **8 KB**
- Each block contains multiple **tuples** (rows).
- Layout inside a page:
    1. **Page header**
    2. **Item pointer array (line pointers)**
    3. **Tuples (actual row data)**

#### Tuple (Item)
- A **tuple** represents a single **row** in a PostgreSQL table.
- Contains:
    - Row data (column values)
    - Tuple header (metadata)
    - Transaction IDs for MVCC (visibility info)
- Tuples can have multiple versions if updated (for concurrency control).

---
### Query Execution Concepts
#### Full Table Scan
- PostgreSQL reads **every row** in a table **from disk into memory** to find matching data.
- Each block of the table is read sequentially from the **heap file**.
- Occurs when:
    - No index exists on the filtered column.
    - The query needs most rows (low selectivity).
    - The planner estimates it’s cheaper than using an index.

#### Index
- An **index** is a separate **data structure** that helps PostgreSQL locate rows **without scanning all data**.
- When using an index scan:
    1. PostgreSQL looks up matching **row pointers** in the index.        
    2. Then it fetches only those specific **heap pages** from **disk into memory**.
- This avoids reading unnecessary blocks.
- Index is automatically generated for primary keys and objects with the `UNIQUE` constraint but are not listed under indexes in PGAdmin.

##### creating an index
```sql
CREATE INDEX ON employees(department);
```
> can but custom name after INDEX

##### dropping an index
```sql
	DROP INDEX <name>;
```

#### `EXPLAIN`
The `EXPLAIN` command shows **how PostgreSQL plans to execute a query** — it does **not actually run** the query.
```sql
EXPLAIN SELECT * FROM users WHERE username = 'ahmed';
```
##### `EXPLAIN ANALYZE`
Executes the query **for real** and displays both:
- The **planner’s estimates** (what PostgreSQL _expected_).
- The **actual runtime statistics** (what _really happened_).
```sql
EXPLAIN ANALYZE SELECT * FROM users WHERE username = 'ahmed';
```
### Inspecting Index & Page Internals
#### Enable Page Inspection
```sql
CREATE EXTENSION pageinspect;
```
- Enables the `pageinspect` extension, which lets you **inspect internal PostgreSQL storage pages**.
- Must be created once per database (superuser privilege required).
- Useful for analyzing **how PostgreSQL physically stores data and index entries** on disk.

### View B-Tree Index Metadata
```sql
SELECT * FROM bt_metap('users_username_idx');
```
- Displays metadata about a **B-Tree index**.

### Inspect Specific Index Pages
```sql
SELECT * FROM bt_page_items('users_username_idx', 3);
```