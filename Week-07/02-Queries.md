For now, we mainly need to understand how to use **SQL queries with SQLx** to read and modify PostgreSQL data.

Think of the flow as:

```text
Rust → SQLx → PostgreSQL
```

And our connection pool is what SQLx uses to communicate with PostgreSQL.

# 1. What is a query?

A query is an SQL instruction sent to the database.

For example:

```sql
SELECT * FROM users;
```

means:

> Give me all users.

Other common queries are:

```sql
INSERT INTO users ...
SELECT ...
UPDATE users ...
DELETE FROM users ...
```

We should know these four very well.

---

# 2. SELECT — Read data

Suppose our database has:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT NOT NULL
);
```

In Rust:

```rust
let users = sqlx::query!(
    "SELECT id, name, email FROM users"
)
.fetch_all(&pool)
.await?;
```

### What's happening?

```rust
sqlx::query!
```

Creates the SQL query.

```rust
.fetch_all(&pool)
```

Executes it using the connection pool and returns **all matching rows**.

```rust
.await
```

waits for PostgreSQL to respond.

---

# 3. Fetch one record

If we want one user:

```rust
let user = sqlx::query!(
    "SELECT id, name, email FROM users WHERE id = $1",
    user_id
)
.fetch_one(&pool)
.await?;
```

The important part is:

```sql
WHERE id = $1
```

and:

```rust
user_id
```

`$1` is a parameter placeholder.

If:

```rust
user_id = 5
```

SQLx effectively sends the value `5` as the parameter.

**Don't build SQL by concatenating user input.**

❌ Avoid:

```rust
format!("SELECT * FROM users WHERE id = {}", user_id)
```

Use parameters:

```rust
sqlx::query!(
    "SELECT * FROM users WHERE id = $1",
    user_id
)
```

---

# 4. `fetch_one` vs `fetch_all`

This is important.

### `fetch_one`

Use when we expect exactly one record:

```rust
.fetch_one(&pool)
```

Example:

```rust
SELECT * FROM users WHERE id = $1
```

If no row exists, it returns an error.

---

### `fetch_all`

Use when we want multiple records:

```rust
.fetch_all(&pool)
```

Example:

```rust
SELECT * FROM users
```

It can return:

```text
User 1
User 2
User 3
```

If there are no matching rows, we get an empty collection.

---

# 5. INSERT — Create data

To create a user:

```rust
let result = sqlx::query!(
    "INSERT INTO users (name, email) VALUES ($1, $2)",
    name,
    email
)
.execute(&pool)
.await?;
```

Notice we're using:

```rust
.execute(&pool)
```

rather than:

```rust
.fetch_all(&pool)
```

because `INSERT` isn't primarily retrieving rows.

---

# 6. INSERT and return the created user

This is very useful for APIs.

```rust
let user = sqlx::query!(
    r#"
    INSERT INTO users (name, email)
    VALUES ($1, $2)
    RETURNING id, name, email
    "#,
    name,
    email
)
.fetch_one(&pool)
.await?;
```

PostgreSQL creates the user and returns the newly created record.

We can then return it from our API.

---

# 7. UPDATE — Modify data

Example:

```rust
sqlx::query!(
    "UPDATE users SET name = $1 WHERE id = $2",
    new_name,
    user_id
)
.execute(&pool)
.await?;
```

This changes the user's name.

For example:

```text
id = 5
name = "Chidiogo"
```

could become:

```text
id = 5
name = "John"
```

---

# 8. DELETE — Remove data

```rust
sqlx::query!(
    "DELETE FROM users WHERE id = $1",
    user_id
)
.execute(&pool)
.await?;
```

This deletes the user whose ID matches `user_id`.

---

# 9. Checking how many rows were affected

`execute()` gives us a result containing information about the operation.

For example:

```rust
let result = sqlx::query!(
    "DELETE FROM users WHERE id = $1",
    user_id
)
.execute(&pool)
.await?;

println!("Deleted {} user(s)", result.rows_affected());
```

If the user doesn't exist:

```text
Deleted 0 user(s)
```

If one was deleted:

```text
Deleted 1 user(s)
```

This is useful when deciding whether our API should return `404 Not Found`.

---

# 10. `query!` vs `query_as!`

For now, understand them like this:

### `query!`

SQLx generates a result based on the selected columns.

```rust
let user = sqlx::query!(
    "SELECT id, name, email FROM users WHERE id = $1",
    user_id
)
.fetch_one(&pool)
.await?;
```

We can access:

```rust
println!("{}", user.id);
println!("{}", user.name);
println!("{}", user.email);
```

---

### `query_as!`

We can map the result into our own Rust struct.

```rust
#[derive(Debug)]
struct User {
    id: i32,
    name: String,
    email: String,
}
```

Then:

```rust
let user = sqlx::query_as!(
    User,
    "SELECT id, name, email FROM users WHERE id = $1",
    user_id
)
.fetch_one(&pool)
.await?;
```

Now:

```rust
println!("{}", user.name);
```

For our backend projects, we will use `query_as!` frequently when working with models.

---

# 11. A complete small example

Assume:

```rust
use sqlx::PgPool;

async fn get_user(
    pool: &PgPool,
    user_id: i32,
) -> Result<(), sqlx::Error> {

    let user = sqlx::query!(
        "SELECT id, name, email FROM users WHERE id = $1",
        user_id
    )
    .fetch_one(pool)
    .await?;

    println!("ID: {}", user.id);
    println!("Name: {}", user.name);
    println!("Email: {}", user.email);

    Ok(())
}
```

The important flow is:

```text
get_user()
    ↓
SQL query
    ↓
$1 ← user_id
    ↓
connection pool
    ↓
PostgreSQL
    ↓
user record
    ↓
Rust
```

---

# 12. The five things we should know

For now, let's make sure we understand these:

| SQL      | SQLx        | Purpose              |
| -------- | ----------- | -------------------- |
| `SELECT` | `fetch_one` | Get one record       |
| `SELECT` | `fetch_all` | Get multiple records |
| `INSERT` | `execute`   | Create data          |
| `UPDATE` | `execute`   | Modify data          |
| `DELETE` | `execute`   | Delete data          |

And understand parameters:

```rust
"SELECT * FROM users WHERE id = $1"
```

```rust
user_id
```

rather than putting user input directly into SQL.

---

## Current goal

We should be comfortable writing something like:

```rust
let users = sqlx::query!(
    "SELECT id, name, email FROM users"
)
.fetch_all(&pool)
.await?;
```

and explaining:

> **SQLx sends the SQL query to PostgreSQL using the connection pool, waits for the result, and gives the returned data back to my Rust program.**

Once we understand that, the next topic should be **database migrations**—creating tables and changing our database structure safely with SQLx.
