# External Crates

For now, we mainly need to understand **what external crates are, how to add them, and how to use them**.

---

## 1. What is an external crate?

An **external crate** is Rust code written by someone else that you add to your project so you don't have to build everything yourself.

For example:

- `tokio` → asynchronous programming
- `axum` → building REST APIs
- `serde` → JSON serialization/deserialization
- `sqlx` → working with databases
- `argon2` → password hashing
- `jsonwebtoken` → JWT authentication

Think of crates like **libraries/packages** in other languages.

For example, in Node.js you might do:

```bash
npm install express
```

In Rust:

```bash
cargo add axum
```

---

# 2. Adding an external crate

Let's create a project:

```bash
cargo new external_crates_demo
cd external_crates_demo
```

Add the `rand` crate:

```bash
cargo add rand
```

Cargo will add it to your `Cargo.toml`.

You might see something like:

```toml
[dependencies]
rand = "0.9"
```

The exact version may differ depending on the current release.

---

# 3. Using the crate

Once installed, import what you need with `use`.

Example:

```rust
use rand::Rng;

fn main() {
    let mut rng = rand::rng();

    let number = rng.random_range(1..=100);

    println!("Random number: {}", number);
}
```

The important pattern is:

```rust
use crate_name::something;
```

For example:

```rust
use rand::Rng;
```

---

# 4. Cargo handles dependencies

You don't manually download and copy crate files into your project.

Cargo handles that for you.

When you run:

```bash
cargo build
```

Cargo:

1. Reads `Cargo.toml`
2. Finds your dependencies
3. Downloads them if necessary
4. Compiles them
5. Compiles your application

So you normally just do:

```bash
cargo add rand
```

and Cargo handles the rest.

---

# 5. `Cargo.toml`

Your dependencies live in:

```toml
[dependencies]
```

Example:

```toml
[package]
name = "my_backend"
version = "0.1.0"
edition = "2024"

[dependencies]
axum = "0.8"
serde = "1"
tokio = "1"
```

This tells Cargo:

> My project depends on Axum, Serde, and Tokio.

---

# 6. Using multiple crates

For your future backend, you might have:

```toml
[dependencies]
axum = "0.8"
tokio = { version = "1", features = ["full"] }
serde = { version = "1", features = ["derive"] }
```

Then in your code:

```rust
use axum::Router;
use serde::{Deserialize, Serialize};
```

Each crate provides functionality for a different job.

---

# 7. Crate vs external crate

You already learned what a **crate** is.

An external crate is simply a crate that comes from outside your project.

For example:

```text
Your project
     │
     ├── Your crate
     │
     └── External crates
            ├── axum
            ├── tokio
            └── serde
```

Your own code:

```rust
fn main() {
    println!("Hello");
}
```

External crate:

```rust
use rand::Rng;
```

---

# 8. `Cargo.lock`

When you add dependencies, Cargo also creates/updates:

```text
Cargo.lock
```

It records the **exact dependency versions** being used.

For example, your `Cargo.toml` might say:

```toml
rand = "0.9"
```

while `Cargo.lock` records the specific version Cargo resolved.

### Important

For an application/backend project, you normally **commit `Cargo.lock` to Git**.

You don't need to memorize how the lock file works. Just understand its purpose.

---

# 9. Features

Some crates have optional functionality called **features**.

For example:

```toml
tokio = { version = "1", features = ["full"] }
```

Here:

```text
tokio
  └── full feature
```

Another common example:

```toml
serde = { version = "1", features = ["derive"] }
```

This enables Serde's derive functionality:

```rust
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize)]
struct User {
    name: String,
}
```

You don't need to learn advanced feature configuration yet. Just recognize the syntax.

---

# 10. Updating crates

You can check outdated dependencies with:

```bash
cargo update
```

And Cargo will resolve newer compatible versions according to your dependency requirements.

You don't need to manually download updates.

---

# 11. Finding crates

The main place to find Rust crates is **crates.io**.

[crates.io](https://crates.io/?utm_source=chatgpt.com)

For example, you can search for:

```text
axum
serde
tokio
sqlx
```

You can also use the documentation site:

[docs.rs](https://docs.rs/?utm_source=chatgpt.com)

For example, when using Axum, you'll frequently look at its documentation to see available types and functions.

---

# 12. A simple practical example

Let's say you want to generate UUIDs for users.

Add:

```bash
cargo add uuid
```

Then:

```rust
use uuid::Uuid;

fn main() {
    let user_id = Uuid::new_v4();

    println!("User ID: {}", user_id);
}
```

The flow is:

```text
cargo add uuid
       ↓
Cargo.toml
       ↓
use uuid::Uuid;
       ↓
Uuid::new_v4()
```

That's the basic workflow you'll use constantly in backend development.

---

# 13. External crates you'll eventually use

For your Rust backend roadmap, these are the important ones to recognize:

| Crate                | Purpose                            |
| -------------------- | ---------------------------------- |
| `tokio`              | Async runtime                      |
| `axum`               | REST APIs                          |
| `serde`              | JSON serialization/deserialization |
| `sqlx`               | PostgreSQL/database access         |
| `uuid`               | UUIDs                              |
| `argon2`             | Password hashing                   |
| `jsonwebtoken`       | JWT                                |
| `validator`          | Request validation                 |
| `tracing`            | Logging                            |
| `tracing-subscriber` | Log configuration                  |

We **don't need to learn all of these now**. We will learn more about them later.

---

# 14. What you should remember for now

If you understand these, you're good:

### External crate

A crate created outside your project that provides reusable functionality.

### Add a crate

```bash
cargo add rand
```

### Dependency

It appears in:

```toml
[dependencies]
rand = "0.9"
```

### Import

```rust
use rand::Rng;
```

### Use

```rust
let mut rng = rand::rng();
let number = rng.random_range(1..=100);
```

### Cargo's job

```text
Cargo.toml
    ↓
Cargo downloads dependencies
    ↓
Cargo compiles dependencies
    ↓
Your application uses them
```

### Most important mental model

> **External crates let you reuse existing Rust libraries instead of writing everything yourself. Cargo manages those dependencies for you.**

That's essentially all you need for **External Crates for now**.
