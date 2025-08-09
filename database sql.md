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
| Constraint      | Description                                     |
| --------------- | ----------------------------------------------- |
| `PRIMARY KEY`   | Unique identifier for each row                  |
| `NOT NULL`      | Column must have a value                        |
| `UNIQUE`        | No duplicate values allowed                     |
| `CHECK (expr)`  | Must satisfy a condition                        |
| `DEFAULT value` | Assigns a default value if none is provided     |
| `REFERENCES`    | Foreign key constraint linking to another table |
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
## Basic SELECT Syntax
```sql
SELECT column1, column2, ...
FROM table_name;
```
## Selecting All Columns
```sql
SELECT * FROM employees;
```
## Filtering Rows (`WHERE`)
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
## Aliases (AS)
```sql
SELECT first_name AS fname, last_name AS lname
FROM employees;
```
## Aggregation Functions
| Function   | Description    |
| ---------- | -------------- |
| `COUNT(*)` | Number of rows |
| `SUM()`    | Sum of values  |
| `AVG()`    | Average value  |
| `MIN()`    | Minimum value  |
| `MAX()`    | Maximum value  |
## String Operations & Functions

| OP/Function | Description                           |
| ----------- | ------------------------------------- |
| \|\|        | Join two strings                      |
| `CONCAT()`  | Join two strings                      |
| `LOWER()`   | Gives a lower case string             |
| `UPPER()`   | Giver a upper case string             |
| `LENGTH()`  | Gives the number of chars in a string |

## Sorting Results (`ORDER BY`)
```sql
SELECT first_name, salary
FROM employees
ORDER BY salary DESC;
```
- `ASC` → ascending (default)
- `DESC` → descending
