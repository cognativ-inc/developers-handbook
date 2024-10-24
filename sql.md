# SQL and PostgreSQL Best Practices

## Avoid SELECT * (Use Explicit Column Names)

**Explanation**:  
Using `SELECT *` retrieves all columns from a table, which can lead to inefficiencies, especially in large tables. It can also make the code harder to maintain since adding or removing columns in the future may cause unexpected issues.

**Bad Example**:
```sql
SELECT * FROM users;
```

**Good Example**:
```sql
SELECT id, first_name, last_name, email FROM users;
```

## Avoid N+1 Query Problem

The N+1 query problem occurs when you perform a query that retrieves a set of records, and for each record, you execute an additional query (hence N additional queries). This leads to performance issues, especially when working with large datasets.

**Bad Example**:
```sql
-- First query retrieves all users
SELECT * FROM users;

-- For each user, a new query is executed to get their orders
SELECT * FROM orders WHERE user_id = 1;
SELECT * FROM orders WHERE user_id = 2;
-- and so on...
```

**Good Example**:
```sql
-- Use a JOIN to retrieve all users and their orders in a single query
SELECT users.id, users.first_name, orders.order_id, orders.amount
FROM users
JOIN orders ON users.id = orders.user_id;
```

## Use Indexes Wisely

Indexes improve query performance but can slow down write operations (inserts, updates). It's important to index the right columns, particularly those used frequently in `WHERE` clauses, `JOINs`, and `ORDER BY` clauses.


**Bad Example**:
```sql
-- No index on frequently queried column
SELECT * FROM orders WHERE customer_id = 123;
```

**Good Example**:
```sql
-- Create an index on customer_id to optimize queries
CREATE INDEX idx_customer_id ON orders (customer_id);

SELECT * FROM orders WHERE customer_id = 123;
```

## Avoid Using `TEXT` When You Can Use VARCHAR

Using TEXT for storing variable-length strings is acceptable, but it can be less performant in certain cases, especially if there are length constraints. VARCHAR allows you to set a length limit, which can help the database optimize storage.

**Bad Example**:
```sql
-- Using TEXT when the data has a defined length limit
CREATE TABLE products (
  name TEXT
);
```

**Good Example**:
```sql
-- Use VARCHAR with a defined limit when possible
CREATE TABLE products (
  name VARCHAR(255)
);
```

## Proper Use of Transactions

Transactions are used to ensure data integrity, particularly when performing multiple related operations. Failing to use transactions can lead to inconsistent data states if one operation in a batch fails.

**Bad Example**:
```sql
-- No transaction used for multiple operations
INSERT INTO accounts (user_id, balance) VALUES (1, 100);
UPDATE users SET status = 'active' WHERE id = 1;
```

**Good Example**:
```sql
-- Using a transaction to ensure both operations succeed or fail together
BEGIN;
INSERT INTO accounts (user_id, balance) VALUES (1, 100);
UPDATE users SET status = 'active' WHERE id = 1;
COMMIT;
```

# Security guidelines

## Avoid SQL Injection by Using Parameterized Queries

SQL Injection occurs when untrusted input is concatenated into SQL queries, allowing attackers to execute malicious code. To prevent this, always use parameterized queries (also known as prepared statements) to safely bind input values.

```sql
-- Vulnerable to SQL injection
SELECT * FROM users WHERE username = '" + userInput + "' AND password = '" + passwordInput + "';
```

**Good Example**:
```sql
-- Using a parameterized query to prevent SQL injection
PREPARE stmt (text, text) AS
SELECT * FROM users WHERE username = $1 AND password = $2;
EXECUTE stmt('john_doe', 'secure_password');
```

## Use Stored Procedures

Stored procedures provide an additional layer of abstraction. They allow the database logic to be defined and executed on the server-side, preventing user inputs from directly altering SQL commands.

**Example:**:

```sql
CREATE OR REPLACE FUNCTION get_user(username text, user_password text)
RETURNS TABLE(id int, name text) AS $$
BEGIN
    RETURN QUERY
    SELECT id, name FROM users WHERE username = $1 AND password = $2;
END;
$$ LANGUAGE plpgsql;

-- Executing the function
SELECT * FROM get_user('john_doe', 'secure_password');
```

## Input Validation and Whitelisting

Another important defense is validating and sanitizing all user inputs. You should always assume that user inputs are untrusted and validate them before using them in a SQL query.

* Use whitelisting: Allow only specific characters (like letters and numbers) or formats for input. Reject anything outside the allowed list.
* Avoid blacklisting, which may be bypassed by sophisticated attacks.

**Example:**:
```sql
-- Only allow usernames with letters, numbers, and underscores
IF userInput ~ '^[A-Za-z0-9_]+$' THEN
    -- safe to proceed
END IF;
```

## Escaping User Input
Escaping is a way to ensure that special characters (such as quotes) in user input are treated as literals rather than part of the SQL code. In PostgreSQL, the `quote_literal()` function helps safely escape user inputs.

* Use whitelisting: Allow only specific characters (like letters and numbers) or formats for input. Reject anything outside the allowed list.
* Avoid blacklisting, which may be bypassed by sophisticated attacks.

**Example:**:
```sql
SELECT * FROM users WHERE username = quote_literal(userInput);
```
However, escaping should not be the primary defense. It is more prone to errors and should only be used as a fallback where parameterized queries are not possible.
