# SQL and PostgreSQL best practices

| Field | Value |
| --- | --- |
| Owner | Engineering |
| Applies to | All Cognativ database work |
| Status | Active |
| Last reviewed | 2026-08-24 |

Examples use PostgreSQL, but the practices apply to any relational database. The mandatory database security requirements are in [Secure Software Development Standard §7](../security/secure-software-development-standard.md#7-database-security).

---

## Contents

**Query practice**

- [Avoid `SELECT *`](#avoid-select-)
- [Avoid the N+1 query problem](#avoid-the-n1-query-problem)
- [Use indexes wisely](#use-indexes-wisely)
- [Choosing string types](#choosing-string-types)
- [Proper use of transactions](#proper-use-of-transactions)

**Security**

- [Prevent SQL injection with parameterized queries](#prevent-sql-injection-with-parameterized-queries)
- [Dynamic identifiers and safe query building](#dynamic-identifiers-and-safe-query-building)
- [Never authenticate inside a SQL query](#never-authenticate-inside-a-sql-query)
- [Stored procedures and functions](#stored-procedures-and-functions)
- [Input validation and allow-listing](#input-validation-and-allow-listing)
- [Escaping is a last resort](#escaping-is-a-last-resort)
- [Least-privileged database credentials](#least-privileged-database-credentials)
- [Credentials never live in source control](#credentials-never-live-in-source-control)
- [Safe migrations](#safe-migrations)
- [Sensitive columns](#sensitive-columns)
- [Production data outside production](#production-data-outside-production)
- [Backups](#backups)

---

## Avoid `SELECT *`

`SELECT *` retrieves every column, which wastes I/O on wide tables and makes code fragile — adding or reordering a column can silently change behavior. It is also a data-exposure risk: a new sensitive column is immediately pulled into every query and, often, into every API response.

**Bad:**

```sql
SELECT * FROM users;
```

**Good:**

```sql
SELECT id, first_name, last_name, email FROM users;
```

List the columns you need. This applies to ORMs too — check what your ORM actually emits, and be explicit about which fields are serialized out of the application.

## Avoid the N+1 query problem

The N+1 problem occurs when you query a set of records and then run one additional query per record. It is usually invisible in development with ten rows and catastrophic in production with ten thousand.

**Bad:**

```sql
-- First query retrieves all users
SELECT id, first_name FROM users;

-- Then one query per user
SELECT order_id, amount FROM orders WHERE user_id = 1;
SELECT order_id, amount FROM orders WHERE user_id = 2;
-- and so on...
```

**Good:**

```sql
-- One query with a join
SELECT users.id, users.first_name, orders.order_id, orders.amount
FROM users
JOIN orders ON users.id = orders.user_id;
```

Alternatively, batch the second query (`WHERE user_id = ANY($1)`), or use your ORM's eager-loading mechanism. Where an ORM is involved, log the generated SQL in development so N+1 patterns are visible.

## Use indexes wisely

Indexes speed up reads and slow down writes. Index the columns you actually filter, join, and sort on — and verify with `EXPLAIN ANALYZE` rather than guessing.

**Bad:**

```sql
-- Frequently filtered column with no supporting index
SELECT id, total FROM orders WHERE customer_id = 123;
```

**Good:**

```sql
CREATE INDEX idx_orders_customer_id ON orders (customer_id);

SELECT id, total FROM orders WHERE customer_id = 123;
```

Notes:

- Foreign keys are not indexed automatically in PostgreSQL. Index them if you query or delete by them.
- A composite index's column order matters; it serves queries that filter on a leading prefix.
- Unused indexes are pure cost. Review them periodically.
- On a live table, create indexes with `CREATE INDEX CONCURRENTLY` to avoid locking out writes.

## Choosing string types

In PostgreSQL, `text`, `varchar`, and `varchar(n)` are stored identically and perform identically — `varchar(n)` simply adds a length check. There is no performance reason to prefer one over the other.

Choose based on whether the length limit is a genuine business rule:

```sql
CREATE TABLE products (
  id          bigserial PRIMARY KEY,
  -- A real constraint: SKUs are always 12 characters
  sku         varchar(12) NOT NULL,
  -- No meaningful upper bound
  description text        NOT NULL
);
```

Do enforce a maximum length at the application boundary for anything user-supplied, so an unbounded input cannot be used to exhaust storage ([standard §6](../security/secure-software-development-standard.md#6-input-validation-and-output-encoding)). Note that changing a `varchar(n)` limit later requires a migration, whereas an application-level bound does not.

> Advice to "avoid `TEXT`, use `VARCHAR` for performance" is a long-standing myth in PostgreSQL. It does not apply. Other engines differ — check your engine's documentation before assuming.

## Proper use of transactions

Wrap related writes in a transaction so they succeed or fail together.

**Bad:**

```sql
-- If the UPDATE fails, the INSERT is still committed
INSERT INTO accounts (user_id, balance) VALUES (1, 100);
UPDATE users SET status = 'active' WHERE id = 1;
```

**Good:**

```sql
BEGIN;
INSERT INTO accounts (user_id, balance) VALUES (1, 100);
UPDATE users SET status = 'active' WHERE id = 1;
COMMIT;
```

Guidelines:

- Keep transactions short. A transaction held open across an HTTP call to a third party holds locks for the duration.
- Never leave a transaction open across a user interaction.
- Handle failure explicitly: roll back and surface a meaningful error ([standard §11](../security/secure-software-development-standard.md#11-error-handling)).
- For read-modify-write sequences on the same row, use `SELECT … FOR UPDATE` or an appropriate isolation level rather than assuming nothing else touched the row.

---

# Security

## Prevent SQL injection with parameterized queries

SQL injection happens when untrusted input becomes part of the SQL statement instead of remaining data. **Always** bind values as parameters. Concatenating or interpolating untrusted values into SQL is prohibited ([standard §6](../security/secure-software-development-standard.md#6-input-validation-and-output-encoding)).

**Vulnerable:**

```sql
-- Untrusted input concatenated into the statement
SELECT id, email FROM users WHERE email = '" + userInput + "';
```

**Good — PostgreSQL prepared statement:**

```sql
PREPARE find_user (text) AS
SELECT id, email, password_hash FROM users WHERE email = $1;

EXECUTE find_user('john_doe@example.com');
```

**Good — from application code (Node.js, `pg`):**

```javascript
const { rows } = await db.query(
  'SELECT id, email, password_hash FROM users WHERE email = $1',
  [email],
);
```

**Good — .NET:**

```csharp
using var command = new NpgsqlCommand(
    "SELECT id, email, password_hash FROM users WHERE email = @email", connection);
command.Parameters.AddWithValue("email", email);
```

Parameters are not string substitution: the value is sent separately from the statement and can never change its structure. This is the primary defense, and it works regardless of what the input contains.

## Dynamic identifiers and safe query building

Parameters bind **values**, not table names, column names, or SQL keywords. You cannot parameterize `ORDER BY $1` or a table name — so those must come from an allow-list in code, never from input:

```javascript
// Map client-supplied sort keys to known-safe SQL
const SORT_COLUMNS = {
  created: 'created_at',
  name: 'last_name',
  total: 'total_amount',
};

function buildOrderBy(sortKey, direction) {
  const column = SORT_COLUMNS[sortKey] ?? 'created_at';
  const dir = direction === 'desc' ? 'DESC' : 'ASC';
  return `ORDER BY ${column} ${dir}`;
}
```

The client can only ever select one of three columns and one of two directions. Where an identifier genuinely must be dynamic inside PL/pgSQL, use `format()` with `%I` (identifier quoting) and `EXECUTE`, never plain concatenation.

Also note: an ORM or query builder is safe only when used as intended. Raw-SQL escape hatches (`query`, `raw`, `whereRaw`, `FromSqlRaw`) reintroduce the full risk, and so do query fragments assembled from strings.

## Never authenticate inside a SQL query

Passing a password to the database to be compared in SQL is wrong for several reasons: it implies the password is stored in a comparable (often plain or reversibly hashed) form, it exposes the password to query logs and monitoring, and the comparison is not constant-time.

**Bad:**

```sql
SELECT id, name FROM users WHERE username = $1 AND password = $2;
```

**Good** — fetch by identifier, then verify the hash in application code:

```sql
SELECT id, email, password_hash FROM users WHERE email = $1;
```

```javascript
const user = rows[0];
// Always run verification, even when no user was found, to avoid leaking
// account existence through response timing.
const ok = user
  ? await argon2.verify(user.password_hash, submittedPassword)
  : await argon2.verify(DUMMY_HASH, submittedPassword).catch(() => false);

if (!ok) {
  return res.status(401).json({ error: 'Invalid credentials' });
}
```

Password storage requirements are in [standard §4](../security/secure-software-development-standard.md#4-authentication); the hashing details are in [general best practices](general-best-practices.md#hash-passwords-properly).

## Stored procedures and functions

Stored procedures and functions are useful for keeping data logic close to the data and for reducing round trips. They are **not** automatically an injection defense: a function that concatenates its arguments into dynamic SQL is exactly as vulnerable as application code that does the same.

```sql
-- Safe: parameters are used as values, no dynamic SQL
CREATE OR REPLACE FUNCTION get_user_by_email(p_email text)
RETURNS TABLE(id bigint, email text) AS $$
BEGIN
    RETURN QUERY
    SELECT u.id, u.email FROM users u WHERE u.email = p_email;
END;
$$ LANGUAGE plpgsql;

SELECT * FROM get_user_by_email('john_doe@example.com');
```

```sql
-- Unsafe: the argument becomes part of the statement
CREATE OR REPLACE FUNCTION search_users(p_filter text)
RETURNS SETOF users AS $$
BEGIN
    RETURN QUERY EXECUTE 'SELECT * FROM users WHERE name = ''' || p_filter || '''';
END;
$$ LANGUAGE plpgsql;
```

If dynamic SQL is unavoidable inside a function, build it with `format()` using `%L` for literals and `%I` for identifiers, and pass values with `EXECUTE … USING`:

```sql
EXECUTE format('SELECT * FROM users WHERE %I = $1', v_column) USING p_value;
```

Also set an explicit `search_path` on `SECURITY DEFINER` functions; without it, a caller can shadow the objects the function references.

## Input validation and allow-listing

Validation is defense in depth on top of parameterization — it is not a substitute for it.

- **Allow-list** what is acceptable: specific characters, formats, enum values, or a bounded pattern. Reject everything else.
- **Do not deny-list.** Blocking `'`, `--`, or `DROP` is bypassable, breaks legitimate input (`O'Brien`), and gives false confidence.
- Validate at the application boundary, ideally with a schema validator, before the value reaches the data layer ([standard §6](../security/secure-software-development-standard.md#6-input-validation-and-output-encoding)).
- Enforce data integrity in the schema as well: `NOT NULL`, foreign keys, `UNIQUE`, `CHECK` constraints, and appropriate types. Constraints catch bugs that application validation misses.

```sql
CREATE TABLE accounts (
  id       bigserial PRIMARY KEY,
  username varchar(30) NOT NULL UNIQUE
           CHECK (username ~ '^[A-Za-z0-9_]{3,30}$'),
  status   text NOT NULL
           CHECK (status IN ('pending', 'active', 'suspended'))
);
```

## Escaping is a last resort

Escaping functions such as `quote_literal()` and `quote_ident()` exist for building dynamic SQL **inside** the database (in PL/pgSQL, typically via `format()`), not for sanitizing input in application code before sending a query.

They must not be your primary defense: they are easy to misuse, easy to forget in one branch of the code, and dependent on getting the quoting context exactly right. Use parameterized queries. Reach for quoting functions only when constructing dynamic SQL server-side, where parameters cannot express what you need.

## Least-privileged database credentials

- The application's database account gets only the privileges the application needs — typically `SELECT`, `INSERT`, `UPDATE`, `DELETE` on its own schema. It should not be a superuser, should not own the schema, and should not have rights over unrelated databases or schemas.
- Run migrations with a separate, higher-privileged account used only for that purpose.
- Use a read-only account for reporting, analytics, and read replicas.
- Give each environment and each service its own credentials.

## Credentials never live in source control

Connection strings, database passwords, and certificates are secrets. They come from the environment or a secret manager at runtime — never from a committed file, a migration, a seed script, or a docker-compose file with real values ([standard §8](../security/secure-software-development-standard.md#8-secrets-management)).

## Safe migrations

- Migrations are version-controlled, reviewed, and reproducible. They run through the deployment pipeline, not by hand on a production console.
- Prefer forward-only migrations in production, with a tested rollback or compensating migration for anything destructive.
- Call out destructive changes explicitly in the pull request: dropping a column or table, narrowing a type, adding a `NOT NULL` to an existing column, or backfilling a large table.
- Use expand/contract for anything that could break a running deployment: add the new column, deploy code that writes both, backfill, switch reads, then remove the old column in a later release.
- Watch for locks. On a large table, adding a column with a volatile default, adding a constraint without `NOT VALID`, or creating an index non-concurrently can block writes for the duration.
- Never put secrets, real user data, or production identifiers in a migration.

## Sensitive columns

- Identify sensitive fields during design — credentials, tokens, government identifiers, health data, financial data — and record where they live and who can read them.
- Keep them out of default reads and default API serializers. This is another reason to avoid `SELECT *`.
- Encrypt or tokenize them where the threat model calls for it ([standard §10](../security/secure-software-development-standard.md#10-encryption)).
- Do not copy them into logs, caches, search indexes, analytics stores, or exports without a documented reason ([standard §9](../security/secure-software-development-standard.md#9-sensitive-data-and-pii)).

## Production data outside production

Production data must not be copied into development, test, demo, or local environments unless it has been anonymized, pseudonymized, or masked, and the copy is authorized by the project lead and permitted by the client's terms. Prefer synthetic fixtures and generated data — they are safer, and they make tests deterministic.

## Backups

Where engineers are responsible for backups: production data stores have backups, backups are encrypted, access to them is restricted to the same standard as the source data, and **restores are tested**. An untested backup is an assumption, not a backup. Retention should match the project's contractual and regulatory requirements ([standard §7](../security/secure-software-development-standard.md#7-database-security)).
