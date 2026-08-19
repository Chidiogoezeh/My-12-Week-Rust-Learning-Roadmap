We mainly need to understand **what traits are, why they exist, how to define them, implement them, and use them as abstractions**.

# 1. What is a Trait?

A **trait defines behavior that a type can have**.

Think of it as a contract:

> "Any type that implements this trait must provide these behaviors."

Example:

```rust
trait Greet {
    fn greet(&self);
}
```

This says:

> Anything implementing `Greet` must have a `greet()` method.

---

# 2. Implementing a Trait

Let's create a `User`:

```rust
struct User {
    name: String,
}
```

Implement `Greet`:

```rust
impl Greet for User {
    fn greet(&self) {
        println!("Hello, {}", self.name);
    }
}
```

Now:

```rust
fn main() {
    let user = User {
        name: String::from("Daud"),
    };

    user.greet();
}
```

Output:

```text
Hello, Daud
```

The relationship is:

```text
Greet trait
    ↓
defines behavior
    ↓
User implements Greet
    ↓
User must provide greet()
```

---

# 3. Why Do We Need Traits?

Imagine you have different types:

```rust
struct User {
    name: String,
}

struct Admin {
    name: String,
}
```

Both should be able to greet.

Instead of writing unrelated functions, define a common trait:

```rust
trait Greet {
    fn greet(&self);
}
```

Then:

```rust
impl Greet for User {
    fn greet(&self) {
        println!("User: {}", self.name);
    }
}

impl Greet for Admin {
    fn greet(&self) {
        println!("Admin: {}", self.name);
    }
}
```

Now both types share the same behavior/interface.

---

# 4. Traits Can Be Used as Function Parameters

This is one of the most important things to understand.

```rust
fn say_hello<T: Greet>(person: &T) {
    person.greet();
}
```

Now both work:

```rust
let user = User {
    name: String::from("Daud"),
};

let admin = Admin {
    name: String::from("John"),
};

say_hello(&user);
say_hello(&admin);
```

The important part:

```rust
T: Greet
```

means:

> `T` can be any type, as long as it implements `Greet`.

---

# 5. `impl Trait` Syntax

Rust gives you a shorter way to write the same idea:

```rust
fn say_hello(person: &impl Greet) {
    person.greet();
}
```

Instead of:

```rust
fn say_hello<T: Greet>(person: &T)
```

For your current level, understand both, but don't worry about advanced generic syntax yet.

---

# 6. Traits Can Have Default Implementations

A trait can provide a default implementation.

```rust
trait Greet {
    fn greet(&self) {
        println!("Hello!");
    }
}
```

Now a type can implement the trait without writing `greet()`:

```rust
struct User;

impl Greet for User {}
```

Then:

```rust
let user = User;
user.greet();
```

Output:

```text
Hello!
```

You can also override the default:

```rust
impl Greet for User {
    fn greet(&self) {
        println!("Hello, User!");
    }
}
```

---

# 7. A Trait Can Have Multiple Methods

```rust
trait Account {
    fn login(&self);
    fn logout(&self);
}
```

Implementing it:

```rust
struct User {
    username: String,
}

impl Account for User {
    fn login(&self) {
        println!("{} logged in", self.username);
    }

    fn logout(&self) {
        println!("{} logged out", self.username);
    }
}
```

Now:

```rust
user.login();
user.logout();
```

---

# 8. Traits + Backend Development

This is where traits become particularly useful for you.

Imagine you have a user repository:

```rust
trait UserRepository {
    fn find_user(&self, id: u32);
    fn create_user(&self);
}
```

You could have a PostgreSQL implementation:

```rust
struct PostgresUserRepository;
```

Then:

```rust
impl UserRepository for PostgresUserRepository {
    fn find_user(&self, id: u32) {
        println!("Finding user {} in PostgreSQL", id);
    }

    fn create_user(&self) {
        println!("Creating user in PostgreSQL");
    }
}
```

Later, you could create a test/mock repository:

```rust
struct MockUserRepository;
```

and:

```rust
impl UserRepository for MockUserRepository {
    fn find_user(&self, id: u32) {
        println!("Mock user {}", id);
    }

    fn create_user(&self) {
        println!("Mock create user");
    }
}
```

Your business logic can depend on the **trait** rather than being tightly coupled to PostgreSQL.

That's a major backend use of traits.

---

# 9. Traits and `derive`

You've probably already seen:

```rust
#[derive(Debug)]
struct User {
    name: String,
}
```

`Debug` is actually a **trait** provided by Rust.

By deriving it, Rust automatically generates an implementation for your struct.

That's why this works:

```rust
println!("{:?}", user);
```

Other traits you'll frequently encounter include:

```rust
#[derive(Debug, Clone, Serialize, Deserialize)]
```

For example, in backend development:

```rust
#[derive(Debug, Serialize, Deserialize)]
struct User {
    id: u32,
    username: String,
}
```

Here:

- `Debug` → allows debugging output
- `Serialize` → Rust → JSON
- `Deserialize` → JSON → Rust

You'll use this constantly with APIs.

---

# 10. Common Traits You Should Know

You don't need to memorize every Rust trait.

Know these first:

### `Debug`

```rust
#[derive(Debug)]
```

Useful for debugging.

---

### `Clone`

```rust
#[derive(Clone)]
```

Allows you to explicitly create a copy of a value when the type supports cloning.

```rust
let user2 = user1.clone();
```

---

### `PartialEq`

```rust
#[derive(PartialEq)]
```

Allows comparisons:

```rust
user1 == user2
```

---

### `Serialize` / `Deserialize`

From Serde:

```rust
#[derive(Serialize, Deserialize)]
```

Extremely important for REST APIs.

---

# 11. Trait Bounds

You'll eventually see:

```rust
fn print_value<T: std::fmt::Debug>(value: T) {
    println!("{:?}", value);
}
```

The:

```rust
T: Debug
```

means:

> `T` must implement `Debug`.

You can have multiple bounds:

```rust
fn process<T: Debug + Clone>(value: T) {
    // ...
}
```

Meaning `T` must implement both:

```text
Debug
+
Clone
```

You don't need to go deeply into trait bounds yet. Just understand what the syntax means.

---

# 12. `impl Trait` as a Return Type

You'll eventually encounter:

```rust
fn create_user() -> impl Greet {
    User {
        name: String::from("Daud"),
    }
}
```

This means:

> This function returns some type that implements `Greet`.

You don't need to master this yet, but you should recognize it.

---

# 13. The Most Important Difference

Don't confuse:

```rust
struct User {
    name: String,
}
```

with:

```rust
trait Greet {
    fn greet(&self);
}
```

A **struct** describes data:

```text
User
 ├── name
 └── email
```

A **trait** describes behavior:

```text
Greet
 └── greet()
```

Then:

```rust
impl Greet for User
```

connects them:

```text
        Greet
          │
       greet()
          │
          ↓
         User
```

---

# What you should master now

Before moving on, make sure you can comfortably:

- Define a trait.
- Implement a trait for a struct.
- Understand trait methods.
- Use `&self` inside trait methods.
- Use default trait implementations.
- Pass trait implementations to functions.
- Understand `T: Trait`.
- Recognize `impl Trait`.
- Understand why `Debug`, `Clone`, `Serialize`, and `Deserialize` are traits.
- Explain how traits can decouple backend business logic from database implementations.

### One backend mental model to remember

```text
Trait
  ↓
defines contract
  ↓
Repository trait
  ↓
PostgresRepository implements it
  ↓
Service uses the contract
  ↓
API handler calls the service
```

That's the level of **Traits** you need for the early part of the Rust backend roadmap.
