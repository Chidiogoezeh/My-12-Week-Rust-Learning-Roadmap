## 1. What is a Package?

A **package** is a Rust project managed by **Cargo**.

When you run:

```bash
cargo new todo_app
```

Cargo creates a package called `todo_app`.

You get:

```text
todo_app/
├── Cargo.toml
└── src/
    └── main.rs
```

The important file is:

```text
Cargo.toml
```

It contains information about your package.

For example:

```toml
[package]
name = "todo_app"
version = "0.1.0"
edition = "2024"

[dependencies]
```

Think of it simply as:

> **Package = the complete project that Cargo manages.**

---

# 2. What is `Cargo.toml`?

`Cargo.toml` is the **manifest file** for your package.

It tells Cargo things such as:

- Package name
- Version
- Rust edition
- Dependencies

Example:

```toml
[package]
name = "todo_app"
version = "0.1.0"
edition = "2024"

[dependencies]
```

When you add a dependency:

```bash
cargo add serde
```

Cargo updates `Cargo.toml`:

```toml
[dependencies]
serde = "1"
```

So you normally **don't need to manually download Rust libraries**.

Cargo manages them for you.

---

# 3. Package vs Crate

This is where confusion usually starts from.

### Package

The **project** managed by Cargo.

```text
todo_app/
├── Cargo.toml
└── src/
    └── main.rs
```

### Crate

A **compilation unit** inside a package.

There are two main types:

- Binary crate → produces an executable
- Library crate → produces a library

For example:

```bash
cargo new my_app
```

creates a package containing a binary crate:

```text
my_app/
├── Cargo.toml
└── src/
    └── main.rs
```

`main.rs` is the root of the binary crate.

---

# 4. Binary Package

When you run:

```bash
cargo new calculator
```

Cargo creates:

```text
calculator/
├── Cargo.toml
└── src/
    └── main.rs
```

`main.rs` contains:

```rust
fn main() {
    println!("Calculator");
}
```

This creates an executable program.

Run it with:

```bash
cargo run
```

---

# 5. Library Package

You can create a library with:

```bash
cargo new math_utils --lib
```

You get:

```text
math_utils/
├── Cargo.toml
└── src/
    └── lib.rs
```

Instead of:

```rust
fn main() {
}
```

you might have:

```rust
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

The `pub` means other crates can access the function.

---

# 6. One Package Can Contain Both

A package can contain:

```text
my_project/
├── Cargo.toml
└── src/
    ├── main.rs
    └── lib.rs
```

In this case, the package contains:

- A binary crate (`main.rs`)
- A library crate (`lib.rs`)

You don't need to use this setup yet. Just understand that **one package can contain multiple crates**.

---

# 7. Dependencies Belong to the Package

Suppose you're building a web API.

You might add:

```bash
cargo add axum
cargo add tokio --features full
cargo add serde --features derive
```

Your `Cargo.toml` becomes something like:

```toml
[package]
name = "my_api"
version = "0.1.0"
edition = "2024"

[dependencies]
axum = "..."
tokio = { version = "...", features = ["full"] }
serde = { version = "...", features = ["derive"] }
```

Cargo downloads and manages those dependencies.

You then use them in your code:

```rust
use axum::Router;
```

---

# 8. Creating a Package

The command you'll use frequently is:

```bash
cargo new project_name
```

Example:

```bash
cargo new task_api
```

Then:

```bash
cd task_api
```

Run:

```bash
cargo run
```

Check your project without running it:

```bash
cargo check
```

Build it:

```bash
cargo build
```

---

# 9. `cargo new` vs `cargo init`

You'll encounter both.

### `cargo new`

Creates a **new directory and package**:

```bash
cargo new my_api
```

Result:

```text
my_api/
├── Cargo.toml
└── src/
    └── main.rs
```

### `cargo init`

Turns an **existing directory into a Rust package**:

```bash
mkdir my_api
cd my_api
cargo init
```

Result:

```text
my_api/
├── Cargo.toml
└── src/
    └── main.rs
```

For now, `cargo new` is the one you'll probably use most.

---

# 10. Package Version

Your `Cargo.toml` might contain:

```toml
[package]
name = "task_api"
version = "0.1.0"
edition = "2024"
```

The version follows semantic versioning:

```text
MAJOR.MINOR.PATCH
```

For example:

```text
1.2.3
│ │ │
│ │ └── patch
│ └──── minor
└────── major
```

You don't need to obsess over versioning yet. Just understand what the three numbers represent.

---

# 11. Package Name

The package name is defined here:

```toml
[package]
name = "task_api"
```

Cargo uses this name when building your project.

For example:

```bash
cargo new task_api
```

creates:

```toml
name = "task_api"
```

---

# 12. Package Structure You'll Eventually Use

As your projects become larger, your package may look like:

```text
task_api/
├── Cargo.toml
├── Cargo.lock
├── src/
│   ├── main.rs
│   ├── routes.rs
│   ├── handlers.rs
│   ├── models.rs
│   └── services.rs
└── tests/
```

At this point, you're dealing with **modules**, which is the next important topic.

For example:

### `src/main.rs`

```rust
mod users;

fn main() {
    users::hello();
}
```

### `src/users.rs`

```rust
pub fn hello() {
    println!("Hello from users");
}
```

Here:

- `task_api` → **package**
- `main.rs` → crate root
- `users.rs` → module
- `hello()` → function

This distinction becomes very important when your backend grows.

---

# The Mental Model To Remember

Don't overcomplicate it.

Think of Rust like this:

```text
PACKAGE
│
├── Cargo.toml
│
├── CRATE
│   └── main.rs
│
├── MODULE
│   └── users.rs
│
└── DEPENDENCIES
    ├── axum
    ├── tokio
    └── serde
```

### In one sentence:

> **A package is a Cargo-managed Rust project; a crate is something Cargo compiles; modules organize the code inside a crate; dependencies are external crates your package uses.**

---

# What is important to know

You should be comfortable with these:

| Concept          | You should know                     |
| ---------------- | ----------------------------------- |
| Package          | ✅ What it is                       |
| `Cargo.toml`     | ✅ What it does                     |
| `cargo new`      | ✅ Create a package                 |
| `cargo init`     | ✅ Initialize an existing directory |
| Binary crate     | ✅ Understand `main.rs`             |
| Library crate    | ✅ Understand `lib.rs`              |
| Dependencies     | ✅ Add/use external crates          |
| Package version  | ✅ Understand `MAJOR.MINOR.PATCH`   |
| `cargo build`    | ✅                                  |
| `cargo run`      | ✅                                  |
| `cargo check`    | ✅                                  |
| Package vs crate | ✅ Basic distinction                |
| Crate vs module  | ✅ Basic distinction                |

You **do not need to memorize Cargo's entire specification** at this stage.

For your backend goal, the most important practical skill is being able to go from:

```bash
cargo new my_api
cd my_api
cargo add axum
cargo add tokio --features full
cargo run
```

to a clean project structure where you can separate your **routes, handlers, models, services, and database code**.
