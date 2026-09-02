# Rust Visibility

**visibility** is mainly about understanding **who can access your functions, structs, fields, and modules**.

The most important rule is:

> **Rust items are private by default. Use `pub` when you want something to be accessible outside its current module.**

---

## 1. Private by default

Consider:

```rust
mod users {
    fn create_user() {
        println!("Creating user");
    }
}

fn main() {
    users::create_user();
}
```

This will **not compile** because `create_user()` is private.

You'll get an error similar to:

```text
error[E0603]: function `create_user` is private
```

---

## 2. Use `pub` to make it accessible

Change it to:

```rust
mod users {
    pub fn create_user() {
        println!("Creating user");
    }
}

fn main() {
    users::create_user();
}
```

Now it works:

```text
Creating user
```

So:

```rust
fn create_user()
```

means **private**.

```rust
pub fn create_user()
```

means **public**.

---

# 3. Visibility with Modules

Suppose you have:

```text
src/
├── main.rs
└── users.rs
```

### `users.rs`

```rust
pub fn create_user() {
    println!("Creating user");
}

fn delete_user() {
    println!("Deleting user");
}
```

### `main.rs`

```rust
mod users;

fn main() {
    users::create_user();
}
```

This works because:

```rust
pub fn create_user()
```

is public.

But:

```rust
users::delete_user();
```

would fail because `delete_user()` is private.

---

# 4. Visibility with Structs

Visibility also applies to structs.

```rust
mod users {
    pub struct User {
        pub name: String,
        pub age: u32,
    }
}
```

Now:

```rust
fn main() {
    let user = users::User {
        name: String::from("Alice"),
        age: 25,
    };

    println!("{}", user.name);
}
```

Everything works because both the struct and its fields are public:

```rust
pub struct User
```

```rust
pub name: String
pub age: u32
```

---

# 5. Public Struct, Private Fields

You can make the struct public while keeping its fields private:

```rust
mod users {
    pub struct User {
        name: String,
        age: u32,
    }
}
```

Then this won't work:

```rust
fn main() {
    let user = users::User {
        name: String::from("Alice"),
        age: 25,
    };
}
```

Why?

Because `User` is public, but:

```rust
name
age
```

are private.

You could instead provide a public function:

```rust
mod users {
    pub struct User {
        name: String,
        age: u32,
    }

    pub fn create_user(name: String, age: u32) -> User {
        User { name, age }
    }
}
```

Then:

```rust
fn main() {
    let user = users::create_user(
        String::from("Alice"),
        25,
    );
}
```

This pattern is useful because the module controls how a `User` is created.

---

# 6. Visibility with Methods

Visibility also applies to methods.

```rust
struct User {
    name: String,
}

impl User {
    pub fn new(name: String) -> User {
        User { name }
    }

    fn secret_method(&self) {
        println!("Private");
    }
}
```

You can do:

```rust
let user = User::new(String::from("Alice"));
```

because `new()` is public.

But you can't call:

```rust
user.secret_method();
```

from outside the appropriate visibility scope because it's private.

---

# 7. Visibility with Modules

Modules themselves can be public.

For example:

```rust
mod users {
    pub mod profile {
        pub fn show() {
            println!("Profile");
        }
    }
}
```

From the appropriate outer scope:

```rust
users::profile::show();
```

Here:

```rust
pub mod profile
```

makes the `profile` module publicly accessible.

---

# 8. `pub` Is Not Just "Global"

One important thing:

`pub` does **not** mean:

> "This can be accessed from absolutely anywhere."

Visibility follows Rust's module structure.

For now, the practical rule you need is:

```text
private
↓
accessible within its allowed module scope

pub
↓
accessible outside that scope according to Rust's visibility rules
```

You don't need to memorize all the advanced visibility rules yet.

---

# 9. Visibility in a Backend

This becomes useful when organizing your backend.

Imagine:

```text
src/
├── main.rs
├── users.rs
└── auth.rs
```

### `users.rs`

```rust
pub struct User {
    pub id: u32,
    pub name: String,
}

pub fn create_user(name: String) -> User {
    User {
        id: 1,
        name,
    }
}
```

### `main.rs`

```rust
mod users;

fn main() {
    let user = users::create_user(
        String::from("Alice")
    );

    println!("{}", user.name);
}
```

The things other parts of your application need are public:

```rust
pub struct User
```

```rust
pub fn create_user()
```

---

# 10. Don't Make Everything `pub`

This is an important habit.

You might be tempted to write:

```rust
pub fn create_user() {}

pub fn validate_user() {}

pub fn hash_password() {}

pub fn internal_logic() {}
```

even when those functions are only needed inside the module.

Don't do that unnecessarily.

Prefer:

```rust
pub fn create_user() {
    validate_user();
    hash_password();
}

fn validate_user() {
    // internal logic
}

fn hash_password() {
    // internal logic
}
```

Now your module exposes only what other parts of your application actually need.

Think of it as:

```text
                 users module
                      │
             ┌────────┴────────┐
             ↓                 ↓
       PUBLIC API          PRIVATE CODE
       create_user()       validate_user()
                           hash_password()
```

This keeps your code easier to maintain.

---

# 11. `pub(crate)` — Know It, But Don't Focus on It Yet

You may eventually see:

```rust
pub(crate) fn create_user() {
}
```

This means the item is public **within the current crate**, but not outside the crate.

For example:

```rust
pub(crate) fn validate_user() {
}
```

This can be useful in larger applications.

For now, just remember:

```text
pub
    → publicly accessible according to module visibility

pub(crate)
    → accessible anywhere within your crate
```

You don't need to use `pub(crate)` regularly yet.

---

# 12. Visibility + `use`

Suppose:

```rust
mod users {
    pub fn create_user() {
        println!("Creating user");
    }
}
```

You can import the function:

```rust
use users::create_user;

fn main() {
    create_user();
}
```

But if you have:

```rust
mod users {
    fn create_user() {
        println!("Creating user");
    }
}
```

then:

```rust
use users::create_user;
```

won't work because the function isn't visible outside the module.

So `use` doesn't bypass visibility.

---

# 13. A Simple Real-World Example

Imagine your future backend has:

```text
src/
├── main.rs
├── users.rs
└── auth.rs
```

`users.rs`:

```rust
pub struct User {
    pub id: u32,
    pub username: String,
}

pub fn find_user(id: u32) -> User {
    User {
        id,
        username: String::from("Alice"),
    }
}

fn validate_user() {
    println!("Validating user");
}
```

The rest of your application can use:

```rust
users::User
```

and:

```rust
users::find_user()
```

but doesn't need direct access to:

```rust
validate_user()
```

That's good visibility design.

---

# What You Should Know After now

| Concept                      | You should know        |
| ---------------------------- | ---------------------- |
| Private by default           | ✅                     |
| `pub`                        | ✅                     |
| Public functions             | ✅                     |
| Private functions            | ✅                     |
| Public structs               | ✅                     |
| Public/private struct fields | ✅                     |
| Public methods               | ✅                     |
| Private methods              | ✅                     |
| Public modules               | ✅                     |
| `use` respects visibility    | ✅                     |
| `pub(crate)`                 | ✅ Basic understanding |

---

## The 4 rules to remember

### Rule 1 — Everything is private by default

```rust
fn hello() {}
```

### Rule 2 — Use `pub` when other modules need access

```rust
pub fn hello() {}
```

### Rule 3 — Struct fields have their own visibility

```rust
pub struct User {
    pub name: String,
    age: u32,
}
```

Here `User` and `name` are public, but `age` is private.

### Rule 4 — Don't make things public unnecessarily

Expose what other parts of your application need; keep implementation details private.
