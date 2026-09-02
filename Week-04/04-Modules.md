# 1. Why do we need modules?

The key idea is:

> **Modules let you organize and control the visibility of your Rust code.**

---

Imagine putting your entire backend inside `main.rs`:

```rust
fn main() {
    // 500 lines of routes
    // 500 lines of database code
    // 500 lines of authentication
    // 500 lines of business logic
}
```

That quickly becomes difficult to maintain.

Instead, we can organize code:

```text
src/
├── main.rs
├── users.rs
├── tasks.rs
└── auth.rs
```

Each file handles a particular part of the application.

---

# 2. Creating a Module

Let's start with a simple example.

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
    println!("Hello from users module");
}
```

Run:

```bash
cargo run
```

Output:

```text
Hello from users module
```

### What's happening?

This:

```rust
mod users;
```

tells Rust:

> "There is a module called `users`."

Then:

```rust
users::hello();
```

means:

> "Call the `hello` function inside the `users` module."

---

# 3. Why `pub`?

Look at:

```rust
pub fn hello() {
    println!("Hello");
}
```

The `pub` means the function is publicly accessible from outside the module.

If you wrote:

```rust
fn hello() {
    println!("Hello");
}
```

then `main.rs` couldn't call it.

So:

```rust
pub fn hello()
```

means:

> This function can be accessed outside this module.

---

# 4. Modules Can Contain Multiple Functions

### `users.rs`

```rust
pub fn create_user() {
    println!("Creating user");
}

pub fn delete_user() {
    println!("Deleting user");
}

pub fn list_users() {
    println!("Listing users");
}
```

### `main.rs`

```rust
mod users;

fn main() {
    users::create_user();
    users::delete_user();
    users::list_users();
}
```

Output:

```text
Creating user
Deleting user
Listing users
```

---

# 5. Modules Can Contain Structs

For example:

### `users.rs`

```rust
pub struct User {
    pub name: String,
    pub age: u32,
}
```

### `main.rs`

```rust
mod users;

fn main() {
    let user = users::User {
        name: String::from("David"),
        age: 25,
    };

    println!("{}", user.name);
}
```

Notice that we have:

```rust
pub struct User
```

and:

```rust
pub name: String
pub age: u32
```

We need `pub` on the fields because we're accessing them from outside the module.

---

# 6. Module Privacy

Rust modules are private by default.

For example:

```rust
mod users {
    fn secret() {
        println!("This is private");
    }
}
```

You can't do:

```rust
users::secret();
```

from outside the module.

But:

```rust
mod users {
    pub fn hello() {
        println!("Hello");
    }
}
```

allows:

```rust
users::hello();
```

### Remember:

```text
private → accessible only where visibility allows

pub → accessible from outside according to Rust's visibility rules
```

This is an important Rust security/design feature.

---

# 7. Nested Modules

Modules can contain other modules.

Example:

```rust
mod users {
    pub mod profile {
        pub fn show() {
            println!("User profile");
        }
    }
}
```

You can access it with:

```rust
fn main() {
    users::profile::show();
}
```

Think of:

```text
users
└── profile
    └── show()
```

The `::` means you're moving through the module path.

---

# 8. `crate::`

You'll see `crate::` frequently in larger Rust projects.

Suppose:

```text
src/
├── main.rs
├── users.rs
└── auth.rs
```

`main.rs`:

```rust
mod users;
mod auth;
```

Inside `auth.rs`, you could refer to something in the root of your crate using:

```rust
crate::users
```

For example:

```rust
crate::users::create_user();
```

`crate::` means:

> Start from the root of this crate.

---

# 9. `super::`

You may also encounter:

```rust
super::
```

It means:

> Go up one module level.

Example:

```rust
mod users {
    pub fn hello() {
        println!("Hello");
    }

    mod profile {
        pub fn show() {
            super::hello();
        }
    }
}
```

`profile` uses:

```rust
super::hello();
```

because `hello()` is one module level above it.

You don't need to use this heavily yet. Just recognize it when you see it.

---

# 10. Organizing a Backend with Modules

This is where modules become really important for you.

Instead of:

```text
src/
└── main.rs
```

your backend might eventually look like:

```text
src/
├── main.rs
├── routes.rs
├── handlers.rs
├── models.rs
├── services.rs
└── db.rs
```

Then:

### `main.rs`

```rust
mod db;
mod handlers;
mod models;
mod routes;
mod services;

fn main() {
    println!("Starting API");
}
```

Each module has a responsibility.

```text
routes.rs
    ↓
API routes

handlers.rs
    ↓
HTTP request handlers

models.rs
    ↓
Data structures

services.rs
    ↓
Business logic

db.rs
    ↓
Database operations
```

This is the beginning of a clean backend architecture.

---

# 11. Example: User Module

Let's make something closer to a real application.

### Structure

```text
src/
├── main.rs
└── users.rs
```

### `users.rs`

```rust
pub struct User {
    pub name: String,
    pub age: u32,
}

pub fn create_user(name: String, age: u32) -> User {
    User {
        name,
        age,
    }
}
```

### `main.rs`

```rust
mod users;

fn main() {
    let user = users::create_user(
        String::from("Alice"),
        25,
    );

    println!("{} is {} years old", user.name, user.age);
}
```

This is much cleaner than putting everything in `main.rs`.

---

# 12. Modules and `use`

You can make module paths shorter with `use`.

Without `use`:

```rust
mod users;

fn main() {
    let user = users::create_user(
        String::from("Alice"),
        25,
    );
}
```

With `use`:

```rust
mod users;

use users::create_user;

fn main() {
    let user = create_user(
        String::from("Alice"),
        25,
    );
}
```

Instead of writing:

```rust
users::create_user()
```

you can simply write:

```rust
create_user()
```

---

# 13. `pub use`

You'll eventually see:

```rust
pub use users::User;
```

This re-exports something.

For example:

```rust
mod users {
    pub struct User;
}

pub use users::User;
```

Now users of your module can access `User` directly instead of:

```rust
users::User
```

This becomes useful in larger libraries, but **you don't need to focus heavily on it yet**.

---

# 14. Modules vs Files

This is an important distinction.

A module doesn't necessarily have to be a separate file.

You can write:

```rust
mod users {
    pub fn hello() {
        println!("Hello");
    }
}

fn main() {
    users::hello();
}
```

Everything is in `main.rs`.

But you can also move it into:

```text
src/users.rs
```

and use:

```rust
mod users;
```

So the file is simply one way of organizing the module.

---

# 15. Real Backend Example

Eventually, you might have:

```text
src/
├── main.rs
├── routes.rs
├── handlers/
│   ├── users.rs
│   └── tasks.rs
├── models/
│   ├── user.rs
│   └── task.rs
└── services/
    ├── auth.rs
    └── task.rs
```

This allows you to organize a serious application.

For example:

```text
routes
 ├── user routes
 └── task routes

handlers
 ├── user handlers
 └── task handlers

models
 ├── User
 └── Task

services
 ├── authentication
 └── task business logic
```

You don't need to build this complicated structure yet. **Start with simple modules and grow the structure when the project actually needs it.**

---

# 16. The `mod` Keyword

There are two common ways you'll see it.

### Inline module

```rust
mod users {
    pub fn hello() {
        println!("Hello");
    }
}
```

### External module

```rust
mod users;
```

with:

```text
src/users.rs
```

Both create a module named `users`.

---

# 17. The Three Keywords You Need

For now, focus heavily on these:

### `mod`

Declares a module:

```rust
mod users;
```

### `pub`

Makes something publicly accessible:

```rust
pub fn create_user() {}
```

### `use`

Brings something into scope:

```rust
use users::create_user;
```

These three will appear constantly in your Rust backend code.

---

# 18. Your Mental Model

Think of your backend like this:

```text
                 YOUR CRATE
                     │
        ┌────────────┼────────────┐
        ↓            ↓            ↓
      routes      handlers      models
        │            │            │
        ↓            ↓            ↓
      routes       logic       structs
```

And Rust gives you:

```rust
mod     // organize code
pub     // control visibility
use     // bring things into scope
```

---

# What You Should Know After now

You should be comfortable with:

| Concept                           | Know it?               |
| --------------------------------- | ---------------------- |
| What a module is                  | ✅                     |
| `mod`                             | ✅                     |
| `pub`                             | ✅                     |
| `use`                             | ✅                     |
| Module paths `::`                 | ✅                     |
| Nested modules                    | ✅                     |
| `crate::`                         | ✅ Basic understanding |
| `super::`                         | ✅ Basic understanding |
| Modules in separate files         | ✅                     |
| Structs inside modules            | ✅                     |
| Functions inside modules          | ✅                     |
| Module privacy                    | ✅                     |
| Organizing a backend with modules | ✅                     |

### Don't worry about yet

You don't need to dive into:

- Complex module trees
- Advanced re-exports
- Cargo workspaces
- Procedural macros
- Advanced visibility patterns
- Complicated library architecture

Those come later when there's a practical reason to use them.

## The key takeaway

Remember this:

```rust
mod users;              // declare module

pub fn create_user() {} // make item public

use users::create_user; // bring item into scope
```

And when you start building your Axum API, you'll use this knowledge to turn:

```text
main.rs
```

from a giant file into organized code such as:

```text
src/
├── main.rs
├── routes.rs
├── handlers.rs
├── models.rs
└── services.rs
```
