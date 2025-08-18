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

### Aliases (AS)

```sql
SELECT first_name AS fname, last_name AS lname
FROM employees AS e;
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

| OP/Function | Description                           |
| ----------- | ------------------------------------- |
| \|\|        | Join two strings                      |
| `CONCAT()`  | Join two strings                      |
| `LOWER()`   | Gives a lower case string             |
| `UPPER()`   | Giver a upper case string             |
| `LENGTH()`  | Gives the number of chars in a string |

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
