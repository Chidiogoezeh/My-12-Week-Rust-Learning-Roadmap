For now, we don't need to learn every detail of connection pooling. We need to understand **what it is, why we need it, and how to use it with Rust + SQLx + PostgreSQL**.

# Connection Pool

## 1. What is a database connection?

Our Rust application needs a connection to PostgreSQL to execute SQL.

Conceptually:

```text
Rust application
       |
       | connection
       ↓
PostgreSQL
```

A connection is like a communication channel between our application and the database.

For example:

```rust
let connection = connect_to_database();
```

Then we can use that connection to run:

```sql
SELECT * FROM users;
```

---

# 2. Why not create a new connection for every request?

Imagine our API receives:

```text
Request 1 → create connection → query → close connection

Request 2 → create connection → query → close connection

Request 3 → create connection → query → close connection
```

Creating a database connection repeatedly is expensive and inefficient.

Instead, we keep a collection of reusable connections.

That's a **connection pool**.

```text
                 ┌── Connection 1 ──┐
                 ├── Connection 2 ──┤
Rust API ────────┼── Connection 3 ──┼──── PostgreSQL
                 ├── Connection 4 ──┤
                 └── Connection 5 ──┘
                     Connection Pool
```

When a request needs the database, it temporarily gets a connection from the pool.

When finished, the connection goes back into the pool.

---

# 3. The important idea

Think of a connection pool as a **waiting room of reusable database connections**.

Instead of:

> "Create a new database connection."

our application says:

> "Give me an available connection from the pool."

This is especially important for web APIs because many requests can arrive at the same time.

---

# 4. SQLx connection pool

With SQLx, we will normally use:

```rust
PgPool
```

`Pg` = PostgreSQL.

First add SQLx:

```bash
cargo add sqlx --features runtime-tokio,tls-rustls,postgres
```

Then:

```rust
use sqlx::postgres::PgPoolOptions;
```

---

# 5. Creating a connection pool

Suppose our PostgreSQL connection string is:

```text
postgres://postgres:password@localhost/myapp
```

We can create a pool:

```rust
use sqlx::postgres::PgPoolOptions;

#[tokio::main]
async fn main() {
    let database_url = "postgres://postgres:password@localhost/myapp";

    let pool = PgPoolOptions::new()
        .max_connections(5)
        .connect(database_url)
        .await
        .unwrap();

    println!("Database connected!");
}
```

The important part is:

```rust
PgPoolOptions::new()
    .max_connections(5)
    .connect(database_url)
    .await
```

We are saying:

> Create a PostgreSQL connection pool that can have up to 5 connections.

---

# 6. Why `async` and `.await`?

Connecting to a database involves waiting for another system.

Rust does not want our application to unnecessarily block while waiting.

That's why:

```rust
.connect(database_url)
```

is asynchronous.

We write:

```rust
.connect(database_url)
.await
```

And our `main` function needs:

```rust
#[tokio::main]
async fn main()
```

We do not need to understand the entire async system yet. For now, remember:

> Database operations with SQLx are asynchronous, so we will commonly use `.await`.

---

# 7. Using the pool to query the database

Suppose we have a `users` table.

We can execute:

```rust
let users = sqlx::query!("SELECT id, name FROM users")
    .fetch_all(&pool)
    .await
    .unwrap();
```

Notice:

```rust
.fetch_all(&pool)
```

We are giving SQLx the pool.

SQLx obtains an available connection from the pool, executes the query, and returns the connection to the pool.

---

# 8. Getting a single record

For example:

```rust
let user = sqlx::query!(
    "SELECT id, name FROM users WHERE id = $1",
    1
)
.fetch_one(&pool)
.await
.unwrap();

println!("{}", user.name);
```

Again:

```rust
.fetch_one(&pool)
```

uses the connection pool.

---

# 9. What does `max_connections(5)` mean?

This:

```rust
.max_connections(5)
```

means:

> The pool can maintain up to 5 database connections.

It **doesn't mean every request gets its own permanent connection**.

For example:

```text
Request 1 ──→ Connection 1
Request 2 ──→ Connection 2
Request 3 ──→ Connection 3
Request 4 ──→ Connection 4
Request 5 ──→ Connection 5
```

If another request arrives while all five are busy:

```text
Request 6
    ↓
waits for an available connection
```

When one becomes available:

```text
Connection 3 becomes available
        ↓
Request 6 gets it
```

---

# 10. Connection pool in an Axum API

This is where it becomes important for our Week 7 project.

We do not want to create a new pool inside every handler.

Don't do this:

```rust
async fn get_users() {
    let pool = PgPoolOptions::new()
        .max_connections(5)
        .connect("...")
        .await
        .unwrap();

    // query...
}
```

Every request would potentially create a new pool.

Instead, create **one pool when our application starts**.

```text
Application starts
       ↓
Create connection pool
       ↓
Start Axum server
       ↓
Requests use the same pool
```

---

# 11. Passing the pool to Axum

A simple example:

```rust
use axum::{extract::State, routing::get, Router};
use sqlx::PgPool;

async fn get_users(
    State(pool): State<PgPool>,
) -> String {
    let _ = pool;

    "Users".to_string()
}

#[tokio::main]
async fn main() {
    let pool = PgPool::connect("postgres://postgres:password@localhost/myapp")
        .await
        .unwrap();

    let app = Router::new()
        .route("/users", get(get_users))
        .with_state(pool);

    // server setup...
}
```

The important pieces are:

### Create the pool:

```rust
let pool = PgPool::connect(DATABASE_URL)
    .await
    .unwrap();
```

### Give it to Axum:

```rust
.with_state(pool)
```

### Receive it in the handler:

```rust
State(pool): State<PgPool>
```

Now our handler can query PostgreSQL through:

```rust
&pool
```

---

# 12. A more realistic handler

For example:

```rust
use axum::extract::State;
use sqlx::PgPool;

async fn get_user(
    State(pool): State<PgPool>,
) -> Result<String, String> {

    let user = sqlx::query!(
        "SELECT name FROM users WHERE id = $1",
        1
    )
    .fetch_one(&pool)
    .await
    .map_err(|error| error.to_string())?;

    Ok(user.name)
}
```

The flow is:

```text
HTTP request
     ↓
Axum handler
     ↓
State<PgPool>
     ↓
SQLx
     ↓
Connection Pool
     ↓
PostgreSQL
     ↓
Result
     ↓
HTTP response
```

That's the basic architecture we need to understand.

---

# 13. What should we remember?

For now, remember these **5 things**:

### ① A database connection

A communication channel between our application and PostgreSQL.

### ② A connection pool

A collection of reusable database connections.

### ③ Why use one?

Because creating database connections repeatedly is expensive.

### ④ Create the pool once

Usually when our application starts:

```rust
let pool = PgPool::connect(DATABASE_URL).await?;
```

### ⑤ Share the pool with our handlers

With Axum:

```rust
.with_state(pool)
```

and:

```rust
State(pool): State<PgPool>
```

---

# Mental model

We should be able to look at this:

```rust
let pool = PgPoolOptions::new()
    .max_connections(10)
    .connect(DATABASE_URL)
    .await?;
```

and understand:

> "I'm creating a reusable pool of up to 10 PostgreSQL connections. My API handlers can use this pool to execute database queries instead of creating a new connection for every request."

**That's enough connection-pool knowledge for now.**

Next , the natural concept to learn is **SQLx queries (`query!`, `query_as!`, `fetch_one`, `fetch_all`, `execute`)**, because that is how we will actually use the connection pool to interact with PostgreSQL.
