Migrations are the next thing we need to understand after connection pools and queries.

The goal is simple:

> **A migration is a version-controlled SQL file that creates or changes your database structure.**

---

# 1. Why do we need migrations?

Suppose your Rust API needs a `users` table.

You could manually open PostgreSQL and run:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT NOT NULL
);
```

But imagine you move your project to another computer.

You'd have to remember:

> "What tables did I create? What columns? What changes did I make?"

Migrations solve this.

You keep the database changes inside your project:

```text
my_api/
├── src/
├── migrations/
│   ├── 202609130001_create_users.sql
│   └── 202609130002_create_todos.sql
└── Cargo.toml
```

Now your database structure is part of your codebase.

---

# 2. SQLx migrations

For your now project, we will use **SQLx migrations**.

You previously installed:

```bash
cargo install sqlx-cli --no-default-features --features postgres
```

Now create a migrations directory:

```bash
sqlx migrate add create_users
```

SQLx creates something similar to:

```text
migrations/
└── 202609130001_create_users.sql
```

The exact timestamp/name will be different on your computer.

---

# 3. Put SQL inside the migration

Open the generated file:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT NOT NULL
);
```

That's your first migration.

It means:

> Create a `users` table with an automatically generated ID, name, and email.

---

# 4. Run the migration

First, make sure PostgreSQL is running and your database exists.

Suppose your database URL is:

```text
postgres://postgres:password@localhost/myapp
```

You can tell SQLx which database to use:

```cmd
set DATABASE_URL=postgres://postgres:password@localhost/myapp
```

Then run:

```bash
sqlx migrate run
```

SQLx executes your migration against PostgreSQL.

Your database now contains:

```text
users
```

---

# 5. What does SQLx actually do?

Think of it like this:

```text
migration file
      ↓
sqlx migrate run
      ↓
PostgreSQL
      ↓
CREATE TABLE users ...
```

SQLx also keeps track of which migrations have already been executed.

So if you run:

```bash
sqlx migrate run
```

again, it won't recreate the `users` table.

It knows:

> "Migration 202609130001 has already been applied."

---

# 6. Adding another table

Later you decide your application needs todos.

Create another migration:

```bash
sqlx migrate add create_todos
```

SQLx creates another file:

```text
migrations/
├── 202609130001_create_users.sql
└── 202609130002_create_todos.sql
```

Put this inside:

```sql
CREATE TABLE todos (
    id SERIAL PRIMARY KEY,
    title TEXT NOT NULL,
    completed BOOLEAN NOT NULL DEFAULT FALSE,
    user_id INTEGER NOT NULL REFERENCES users(id)
);
```

Then:

```bash
sqlx migrate run
```

Now you have:

```text
users
  │
  └── todos
```

The `user_id` establishes a relationship between a todo and its user.

---

# 7. Migration order matters

Imagine:

```text
001_create_users.sql
002_create_todos.sql
```

The users table should be created first because `todos` contains:

```sql
REFERENCES users(id)
```

So SQLx applies migrations in order:

```text
001 → 002 → 003 → ...
```

Don't randomly change the order of migrations that have already been applied.

---

# 8. What if you need to change a table?

Suppose you originally created:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT NOT NULL
);
```

Later you want to add:

```text
created_at
```

**Don't edit the old migration.**

Create a new migration:

```bash
sqlx migrate add add_created_at_to_users
```

Then:

```sql
ALTER TABLE users
ADD COLUMN created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP;
```

Run:

```bash
sqlx migrate run
```

Now your migration history is:

```text
001_create_users
002_create_todos
003_add_created_at_to_users
```

This is the important habit:

> **Once a migration has been applied, create a new migration for future changes instead of editing the old one.**

---

# 9. Rolling back a migration

SQLx supports reverting the most recent migration:

```bash
sqlx migrate revert
```

For example:

```text
001_create_users
002_create_todos
003_add_created_at
```

If you run:

```bash
sqlx migrate revert
```

SQLx reverts the latest migration.

For that reason, migrations can have an `up` and `down` concept in migration systems, although with SQLx's standard migration files you'll typically write the SQL needed for the forward change and use SQLx's migration tooling during development.

For your beginner project, don't overcomplicate this. Focus on **creating and running migrations**.

---

# 10. Migration vs Query

This distinction is important.

### Migration

Changes the **structure of the database**.

Examples:

```sql
CREATE TABLE users (...);
```

```sql
ALTER TABLE users ADD COLUMN age INTEGER;
```

```sql
CREATE INDEX ...
```

### Query

Works with the **data** inside the database.

Examples:

```sql
SELECT * FROM users;
```

```sql
INSERT INTO users (...);
```

```sql
UPDATE users SET ...;
```

```sql
DELETE FROM users WHERE ...;
```

Think:

```text
Migration → Change database structure

Query → Work with database data
```

---

# 11. Week 7 project

For your Todo API, you should be able to create these migrations.

### Migration 1

```bash
sqlx migrate add create_users
```

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT NOT NULL UNIQUE
);
```

### Migration 2

```bash
sqlx migrate add create_todos
```

```sql
CREATE TABLE todos (
    id SERIAL PRIMARY KEY,
    title TEXT NOT NULL,
    completed BOOLEAN NOT NULL DEFAULT FALSE,
    user_id INTEGER NOT NULL REFERENCES users(id)
);
```

Then:

```bash
sqlx migrate run
```

Your database becomes:

```text
users
----------------
id
name
email

        │
        │ user_id
        ↓

todos
----------------
id
title
completed
user_id
```

Your Rust API can then use the **connection pool + queries** you learned earlier to work with these tables.

---

# 12. The complete Week 7 picture

You should understand how these three pieces fit together:

```text
             PostgreSQL
                 ↑
                 │
          ┌──────┴──────┐
          │             │
     Migrations       Queries
          ↑             ↑
          │             │
          └──── SQLx ───┘
                 ↑
                 │
            Connection
               Pool
                 ↑
                 │
             Rust API
```

### Migration:

> "Create/change my database structure."

### Query:

> "Read or modify data in my database."

### Connection pool:

> "Manage reusable connections between my API and PostgreSQL."

---

## What you should know by the end of Week 7

You don't need more than these **6 things** right now:

1. **What a migration is**
2. How to create one:

   ```bash
   sqlx migrate add create_users
   ```

3. How to write SQL inside it:

   ```sql
   CREATE TABLE users (...);
   ```

4. How to apply it:

   ```bash
   sqlx migrate run
   ```

5. Why you **don't edit an already-applied migration**
6. The difference between **migrations and queries**

If you understand those, you are now ready to build the **Todo REST API with Axum + PostgreSQL + SQLx**.
