# Enums

## 1. What is an Enum?

An **enum** allows you to define a type that can have **one of several possible variants**.

For example, a user can have one of three roles:

```rust
enum Role {
    Admin,
    User,
    Guest,
}
```

A variable of type `Role` can be **one of those variants**:

```rust
let role = Role::Admin;
```

Think:

```text
Role
 ├── Admin
 ├── User
 └── Guest
```

This is useful when something has a **fixed set of possible states**.

---

# 2. Using `match`

The most important thing to learn with enums is `match`.

```rust
enum Role {
    Admin,
    User,
    Guest,
}

fn main() {
    let role = Role::Admin;

    match role {
        Role::Admin => println!("Full access"),
        Role::User => println!("Normal access"),
        Role::Guest => println!("Read-only access"),
    }
}
```

Output:

```text
Full access
```

### Why `match` matters

Rust requires you to handle **every possible variant**.

If you add:

```rust
enum Role {
    Admin,
    User,
    Guest,
    Moderator,
}
```

your `match` must account for `Moderator` too.

This makes Rust code safer.

---

# 3. Enums Can Contain Data

This is where enums become particularly powerful.

```rust
enum UserStatus {
    Active,
    Suspended(String),
    Banned(String),
}
```

Now:

```rust
let status = UserStatus::Suspended(
    String::from("Violation of community rules")
);
```

You can extract the data with `match`:

```rust
match status {
    UserStatus::Active => {
        println!("User is active");
    }

    UserStatus::Suspended(reason) => {
        println!("Suspended: {}", reason);
    }

    UserStatus::Banned(reason) => {
        println!("Banned: {}", reason);
    }
}
```

The variants don't have to contain the same type or amount of data.

---

# 4. Different Variants Can Have Different Data

For example:

```rust
enum Payment {
    Cash,
    Card(String),
    BankTransfer {
        account_number: String,
        bank: String,
    },
}
```

Now:

```rust
let payment = Payment::Card(String::from("****1234"));
```

Or:

```rust
let payment = Payment::BankTransfer {
    account_number: String::from("1234567890"),
    bank: String::from("Example Bank"),
};
```

This is one of the major differences between enums and structs.

### Struct

A struct contains **all of its fields**:

```rust
struct User {
    id: u32,
    name: String,
    email: String,
}
```

### Enum

An enum contains **one variant at a time**:

```rust
enum Payment {
    Cash,
    Card(String),
    BankTransfer {
        account_number: String,
        bank: String,
    },
}
```

---

# 5. Enums + `match` Are Extremely Useful for Backend States

Suppose you're processing an order:

```rust
enum OrderStatus {
    Pending,
    Processing,
    Shipped,
    Delivered,
    Cancelled,
}
```

Then:

```rust
fn describe_status(status: OrderStatus) {
    match status {
        OrderStatus::Pending => println!("Waiting for processing"),
        OrderStatus::Processing => println!("Being processed"),
        OrderStatus::Shipped => println!("On the way"),
        OrderStatus::Delivered => println!("Delivered"),
        OrderStatus::Cancelled => println!("Order cancelled"),
    }
}
```

This pattern appears everywhere in backend development.

---

# 6. Enums + `impl`

Just like structs, enums can have methods.

```rust
enum Role {
    Admin,
    User,
    Guest,
}

impl Role {
    fn can_delete_users(&self) -> bool {
        match self {
            Role::Admin => true,
            Role::User => false,
            Role::Guest => false,
        }
    }
}
```

Use it:

```rust
let role = Role::Admin;

if role.can_delete_users() {
    println!("You can delete users");
}
```

---

# 7. The `Option` Enum

This is **one of the most important enums in Rust**.

You should understand it very well.

Rust's `Option` essentially looks like:

```rust
enum Option<T> {
    Some(T),
    None,
}
```

It represents:

> "There might be a value, or there might not be."

Example:

```rust
let username: Option<String> = Some(String::from("daud"));
```

Or:

```rust
let username: Option<String> = None;
```

Instead of returning a potentially dangerous `null`, Rust makes the possibility explicit.

---

## Using `match` with `Option`

```rust
let username = Some(String::from("daud"));

match username {
    Some(name) => println!("Username: {}", name),
    None => println!("No username found"),
}
```

This is extremely common when dealing with databases.

For example:

```rust
fn find_user(id: u32) -> Option<String> {
    if id == 1 {
        Some(String::from("daud"))
    } else {
        None
    }
}
```

Then:

```rust
match find_user(1) {
    Some(name) => println!("Found: {}", name),
    None => println!("User not found"),
}
```

---

# 8. The `Result` Enum

For backend development, **`Result` is arguably even more important**.

Conceptually:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

It represents an operation that can either:

```text
success → Ok
failure → Err
```

Example:

```rust
fn divide(a: f64, b: f64) -> Result<f64, String> {
    if b == 0.0 {
        Err(String::from("Cannot divide by zero"))
    } else {
        Ok(a / b)
    }
}
```

Use it:

```rust
match divide(10.0, 2.0) {
    Ok(result) => println!("Result: {}", result),
    Err(error) => println!("Error: {}", error),
}
```

Output:

```text
Result: 5
```

---

# 9. `Result` in Backend Development

Imagine your API tries to find a user:

```rust
fn get_user(id: u32) -> Result<String, String> {
    if id == 1 {
        Ok(String::from("Daud"))
    } else {
        Err(String::from("User not found"))
    }
}
```

Then:

```rust
match get_user(10) {
    Ok(user) => println!("User: {}", user),
    Err(error) => println!("Error: {}", error),
}
```

This pattern is fundamental to Rust backend programming.

You'll use `Result` for:

- Database queries
- Authentication
- File operations
- HTTP requests
- Parsing
- Validation
- Configuration
- API handlers

---

# 10. The `?` Operator

Once you understand `Result`, learn `?`.

Instead of:

```rust
fn get_user() -> Result<String, String> {
    let result = find_user();

    match result {
        Ok(user) => Ok(user),
        Err(error) => Err(error),
    }
}
```

You can often write:

```rust
fn get_user() -> Result<String, String> {
    let user = find_user()?;

    Ok(user)
}
```

The `?` essentially says:

> If this succeeded, give me the value. If it failed, return the error from this function.

You'll use this **constantly** in backend Rust.

---

# 11. Enums in REST APIs

Imagine your API has authentication states:

```rust
enum AuthStatus {
    Authenticated,
    Unauthenticated,
    Expired,
}
```

Then:

```rust
match status {
    AuthStatus::Authenticated => {
        println!("Allow request");
    }

    AuthStatus::Unauthenticated => {
        println!("Return 401");
    }

    AuthStatus::Expired => {
        println!("Token expired");
    }
}
```

This is much safer than passing around arbitrary strings like:

```rust
"authenticated"
"unauthenticated"
"expired"
```

because the compiler knows exactly what values are valid.

---

# 12. Enums + Structs

You'll frequently combine them.

```rust
enum Role {
    Admin,
    User,
}

struct User {
    id: u32,
    username: String,
    role: Role,
}
```

Create a user:

```rust
let user = User {
    id: 1,
    username: String::from("daud"),
    role: Role::Admin,
};
```

Then:

```rust
match user.role {
    Role::Admin => println!("Administrator"),
    Role::User => println!("Regular user"),
}
```

This combination is extremely common in real applications.

---

# 13. Enums + `derive`

Just like structs, enums can derive traits.

```rust
#[derive(Debug)]
enum Role {
    Admin,
    User,
    Guest,
}
```

Now:

```rust
println!("{:?}", Role::Admin);
```

Output:

```text
Admin
```

You'll commonly see:

```rust
#[derive(Debug, Clone)]
enum Role {
    Admin,
    User,
    Guest,
}
```

Later, you'll also encounter derives related to serialization/deserialization:

```rust
#[derive(Debug, Serialize, Deserialize)]
enum Role {
    Admin,
    User,
}
```

That's particularly relevant when working with APIs.

---

# 14. A Backend Example

Here's a small example combining what you've learned:

```rust
#[derive(Debug)]
enum Role {
    Admin,
    User,
}

#[derive(Debug)]
struct UserAccount {
    id: u32,
    username: String,
    role: Role,
}

impl UserAccount {
    fn can_delete_user(&self) -> bool {
        match self.role {
            Role::Admin => true,
            Role::User => false,
        }
    }
}

fn main() {
    let user = UserAccount {
        id: 1,
        username: String::from("daud"),
        role: Role::Admin,
    };

    println!("{:?}", user);

    if user.can_delete_user() {
        println!("Permission granted");
    } else {
        println!("Permission denied");
    }
}
```

Notice how several Rust concepts are working together:

```text
Struct
  ↓
UserAccount

Enum
  ↓
Role

impl
  ↓
can_delete_user()

match
  ↓
determine permission
```

That's much closer to the kind of Rust you'll eventually write in your backend projects.

---

# What You Should Know Before Moving On

You should be comfortable with:

### Essential

- What an enum is
- Creating enum variants
- Using `match`
- Matching every variant
- Enums containing data
- `impl` with enums

### **Very important for backend**

- `Option<T>` → value may or may not exist
- `Result<T, E>` → operation succeeded or failed
- `Ok(...)`
- `Err(...)`
- `Some(...)`
- `None`
- The `?` operator

### Later

- Enums + Serde
- Pattern matching techniques
- Generic enums
- Custom error enums
- Advanced pattern matching

---

## The mental model

Remember these three:

```text
STRUCT
"What data does this thing have?"

ENUM
"What possible forms/states can this thing have?"

MATCH
"What should I do for each possible form/state?"
```

For backend development, you'll constantly encounter patterns like:

```rust
struct User {
    id: u32,
    role: Role,
}

enum Role {
    Admin,
    User,
}

fn find_user(id: u32) -> Result<User, String> {
    // ...
}
```

That's the combination of **Structs + Enums + Result** that you should aim to master before moving further into Axum and PostgreSQL.
