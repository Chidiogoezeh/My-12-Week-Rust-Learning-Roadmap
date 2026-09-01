## 1. What is a Crate?

A **crate is a unit of compilation in Rust**.

In simple terms:

> A crate is a piece of Rust code that the compiler treats as one unit.

There are two main types:

1. **Binary crate** → creates an executable program.
2. **Library crate** → provides code that other code can use.

---

# 2. Binary Crate

A binary crate is a program that you can run.

When you create a normal Cargo project:

```bash
cargo new calculator
```

you get:

```text
calculator/
├── Cargo.toml
└── src/
    └── main.rs
```

`main.rs` is the **crate root** of the binary crate.

```rust
fn main() {
    println!("Hello, Rust!");
}
```

Run it:

```bash
cargo run
```

Output:

```text
Hello, Rust!
```

So:

```text
main.rs
   ↓
Binary crate
   ↓
Executable program
```

---

# 3. Library Crate

A library crate is code designed to be **used by other Rust code**.

Create one:

```bash
cargo new math_utils --lib
```

You'll get:

```text
math_utils/
├── Cargo.toml
└── src/
    └── lib.rs
```

`lib.rs` is the crate root.

Put this inside:

```rust
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

The `pub` keyword means the function can be accessed from outside the module/crate.

---

# 4. Binary Crate vs Library Crate

The easiest way to remember the difference:

```text
Binary crate
    ↓
main.rs
    ↓
Runs a program
```

```text
Library crate
    ↓
lib.rs
    ↓
Provides reusable code
```

For example:

```text
Application → binary crate
Reusable functionality → library crate
```

---

# 5. A Package Can Contain Crates

This is where **package vs crate** becomes important.

A **package** is the project managed by Cargo.

A package can contain one or more crates.

For example:

```text
my_project/
├── Cargo.toml
└── src/
    ├── main.rs
    └── lib.rs
```

This package contains:

```text
my_project
    │
    ├── binary crate
    │      └── main.rs
    │
    └── library crate
           └── lib.rs
```

You don't need to use both in every project.

---

# 6. External Crates

You'll frequently use crates created by other Rust developers.

For example, for backend development we'll use:

- `axum` → web framework
- `tokio` → asynchronous runtime
- `serde` → serialization/deserialization
- `sqlx` → database access
- `jsonwebtoken` → JWT handling

You add them to your project with Cargo.

For example:

```bash
cargo add axum
```

Cargo adds it to:

```toml
[dependencies]
axum = "..."
```

Then you can use it:

```rust
use axum::Router;

fn main() {
    let app = Router::new();

    println!("API created");
}
```

You don't need to manually download the crate.

**Cargo handles that.**

---

# 7. Using a Crate with `use`

Suppose you add:

```bash
cargo add serde --features derive
```

You can then import things from the crate:

```rust
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize)]
struct User {
    name: String,
    age: u32,
}
```

Here:

```text
serde
  ↓
crate

Serialize
Deserialize
  ↓
items we're importing
```

---

# 8. Crate Names and `use`

Consider:

```rust
use axum::Router;
```

Break it down:

```text
axum
 │
 └── crate

Router
 │
 └── item inside the crate
```

Another example:

```rust
use std::collections::HashMap;
```

Here `std` is Rust's standard library crate, and `HashMap` is something inside it.

---

# 9. The Standard Library

Rust comes with a standard library called:

```text
std
```

You don't need to install it.

For example:

```rust
use std::collections::HashMap;

fn main() {
    let mut users = HashMap::new();

    users.insert(1, "Alice");
    users.insert(2, "Bob");

    println!("{:?}", users);
}
```

You're using functionality from the standard library crate.

---

# 10. Creating Your Own Library Crate

Let's make a small example.

Create:

```bash
cargo new calculator --lib
```

`src/lib.rs`:

```rust
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}

pub fn subtract(a: i32, b: i32) -> i32 {
    a - b
}
```

Now your library provides:

```text
add()
subtract()
```

to code that uses the crate.

---

# 11. Crates and Modules

Don't confuse **crates** with **modules**.

Suppose you have:

```text
src/
├── main.rs
├── users.rs
└── tasks.rs
```

You have **one crate** with multiple modules:

```text
Binary crate
│
├── users module
└── tasks module
```

For example:

### `main.rs`

```rust
mod users;
mod tasks;

fn main() {
    users::hello();
    tasks::hello();
}
```

### `users.rs`

```rust
pub fn hello() {
    println!("Users module");
}
```

### `tasks.rs`

```rust
pub fn hello() {
    println!("Tasks module");
}
```

The important distinction:

> **Crate = compilation unit.**
> **Module = way of organizing code inside a crate.**

---

# 12. Crates in a Backend Project

Eventually your backend might look like:

```text
task_api/
│
├── Cargo.toml
│
└── src/
    ├── main.rs
    ├── routes.rs
    ├── handlers.rs
    ├── models.rs
    └── services.rs
```

This is generally **one binary crate**.

But that crate uses other crates:

```text
task_api
   │
   ├── axum
   ├── tokio
   ├── serde
   ├── sqlx
   └── tracing
```

Your project is the crate you're developing, while those are external crates your project depends on.

---

# 13. Where Cargo Fits

You can think of the relationship like this:

```text
Cargo
  │
  ↓
Package
  │
  ├── Your crate
  │
  └── Dependencies
       ├── axum
       ├── tokio
       ├── serde
       └── sqlx
```

Cargo handles:

- Downloading dependencies
- Compiling crates
- Managing versions
- Building your project
- Running your program

---

# 14. `Cargo.toml` and Crates

Suppose your `Cargo.toml` contains:

```toml
[dependencies]
axum = "..."
tokio = { version = "...", features = ["full"] }
serde = { version = "...", features = ["derive"] }
```

Those dependencies are crates that your project uses.

Then in your Rust code:

```rust
use axum::Router;
use serde::{Deserialize, Serialize};
```

You're importing functionality from those crates.

---

# 15. One Important Concept: `pub`

You'll encounter this constantly when working with crates and modules.

Without `pub`:

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

The function isn't publicly accessible outside its module.

With:

```rust
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

it can be accessed from outside.

You don't need to master Rust's entire visibility system yet. Just understand:

> `pub` makes an item publicly accessible according to Rust's module visibility rules.

---

# 16. What To Know

You should be comfortable with:

| Concept          | What you should know                         |
| ---------------- | -------------------------------------------- |
| Crate            | ✅ Unit of compilation                       |
| Binary crate     | ✅ Uses `main.rs` and produces an executable |
| Library crate    | ✅ Uses `lib.rs` and provides reusable code  |
| Crate root       | ✅ `main.rs` or `lib.rs`                     |
| External crate   | ✅ Reusable code from another project        |
| `use`            | ✅ Import functionality                      |
| `pub`            | ✅ Make items publicly accessible            |
| `std`            | ✅ Rust's standard library                   |
| Package vs crate | ✅ Basic distinction                         |
| Crate vs module  | ✅ Basic distinction                         |
| Cargo            | ✅ Manages packages and dependencies         |

---

## The 3 things I want you to remember

If you remember nothing else from this lesson, remember these:

### 1. A crate is a compilation unit

```text
main.rs → binary crate
lib.rs  → library crate
```

### 2. A package can contain crates

```text
Package
  ├── Binary crate
  └── Library crate
```

### 3. Your backend will use many crates

```text
Your API
   │
   ├── axum
   ├── tokio
   ├── serde
   ├── sqlx
   └── tracing
```

That's enough **crate knowledge for now**. Don't go down the rabbit hole of Cargo workspaces, publishing crates, procedural macros, crate registries, or advanced package configuration yet. Those become useful later when you actually need them.
