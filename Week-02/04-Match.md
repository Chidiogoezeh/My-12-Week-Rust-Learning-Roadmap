Here we understand **what Match does, how patterns work, how it returns values, and how it works with `Option` and `Result`**.

# 1. What is `match`?

`match` lets you compare a value against several possible patterns and execute the matching branch.

Think:

> "Given this value, what should I do?"

Basic example:

```rust
fn main() {
    let number = 2;

    match number {
        1 => println!("One"),
        2 => println!("Two"),
        3 => println!("Three"),
        _ => println!("Something else"),
    }
}
```

Output:

```text
Two
```

`_` means **anything else**.

---

# 2. Basic Syntax

```rust
match value {
    pattern => expression,
    pattern => expression,
    _ => expression,
}
```

For example:

```rust
let age = 20;

match age {
    18 => println!("You are 18"),
    20 => println!("You are 20"),
    _ => println!("Another age"),
}
```

---

# 3. `match` Must Be Exhaustive

Rust wants you to handle **every possible case**.

This will cause an error:

```rust
let number = 5;

match number {
    1 => println!("One"),
    2 => println!("Two"),
}
```

Why?

Because what happens if `number` is `3`, `4`, `5`, etc.?

You can handle the remaining cases with `_`:

```rust
match number {
    1 => println!("One"),
    2 => println!("Two"),
    _ => println!("Something else"),
}
```

This is one of the most important things to understand about `match`.

---

# 4. `match` Can Return a Value

This is extremely useful.

```rust
let number = 2;

let result = match number {
    1 => "One",
    2 => "Two",
    3 => "Three",
    _ => "Unknown",
};

println!("{}", result);
```

Output:

```text
Two
```

Notice there's no semicolon after the expressions:

```rust
2 => "Two",
```

The `match` itself produces a value.

---

# 5. Multiple Patterns

You can match multiple values in one branch:

```rust
let number = 2;

match number {
    1 | 2 | 3 => println!("Small number"),
    4 | 5 | 6 => println!("Medium number"),
    _ => println!("Other"),
}
```

`|` means **OR**.

---

# 6. Ranges

You can match ranges:

```rust
let age = 25;

match age {
    0..=12 => println!("Child"),
    13..=17 => println!("Teenager"),
    18..=64 => println!("Adult"),
    65.. => println!("Senior"),
}
```

`..=` means the range includes the ending value.

So:

```rust
18..=64
```

means:

```text
18 through 64
```

---

# 7. Matching Enums

This is where `match` becomes particularly important in Rust.

Suppose:

```rust
enum Status {
    Active,
    Inactive,
    Suspended,
}
```

You can match it:

```rust
let status = Status::Active;

match status {
    Status::Active => println!("User is active"),
    Status::Inactive => println!("User is inactive"),
    Status::Suspended => println!("User is suspended"),
}
```

This is one reason **Enums + Match** are commonly taught together.

---

# 8. Matching Enums with Data

Enums can contain data.

```rust
enum Payment {
    Cash,
    Card { amount: f64 },
    Transfer { amount: f64 },
}
```

Now:

```rust
let payment = Payment::Card { amount: 500.0 };

match payment {
    Payment::Cash => println!("Cash payment"),

    Payment::Card { amount } => {
        println!("Card payment: {}", amount);
    }

    Payment::Transfer { amount } => {
        println!("Transfer: {}", amount);
    }
}
```

The `match` can **extract the data** from the enum.

This pattern is extremely important in real Rust applications.

---

# 9. `match` with `Option`

You'll use this frequently in backend Rust.

`Option` represents:

```text
Some(value)
or
None
```

Example:

```rust
let username: Option<String> = Some(String::from("daud"));

match username {
    Some(name) => println!("Username: {}", name),
    None => println!("No username found"),
}
```

If it's:

```rust
let username: Option<String> = None;
```

then:

```text
No username found
```

---

# 10. `match` with `Result`

This is **very important for backend development**.

`Result` represents:

```text
Ok(value)
or
Err(error)
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

Then:

```rust
let result = divide(10.0, 2.0);

match result {
    Ok(value) => println!("Result: {}", value),
    Err(error) => println!("Error: {}", error),
}
```

Output:

```text
Result: 5
```

With:

```rust
let result = divide(10.0, 0.0);
```

you get:

```text
Error: Cannot divide by zero
```

You'll use this concept constantly when dealing with:

- Database operations
- HTTP requests
- Authentication
- File operations
- Parsing
- Validation

---

# 11. Matching and Returning Errors

A backend-style example:

```rust
fn find_user(id: u32) -> Result<String, String> {
    if id == 1 {
        Ok(String::from("Daud"))
    } else {
        Err(String::from("User not found"))
    }
}

fn main() {
    match find_user(2) {
        Ok(username) => println!("Found: {}", username),
        Err(error) => println!("Error: {}", error),
    }
}
```

This pattern will become very familiar when you start using SQLx.

---

# 12. Matching with Conditions

You can add a condition using a **guard**:

```rust
let number = 10;

match number {
    n if n > 0 => println!("Positive"),
    n if n < 0 => println!("Negative"),
    _ => println!("Zero"),
}
```

The:

```rust
if n > 0
```

is called a **match guard**.

You don't need to master complex guards yet.

---

# 13. `match` vs `if`

Use `if` when you're mainly checking conditions:

```rust
if age >= 18 {
    println!("Adult");
} else {
    println!("Minor");
}
```

Use `match` when you're dealing with **multiple distinct patterns or states**:

```rust
match status {
    Status::Active => println!("Active"),
    Status::Inactive => println!("Inactive"),
    Status::Suspended => println!("Suspended"),
}
```

A useful rule:

> **`if` → conditions.**
> **`match` → patterns and states.**

There are exceptions, but this is a good beginner mental model.

---

# 14. `match` in Your CLI Calculator

This connects directly to the calculator you built earlier.

Instead of:

```rust
if choice == 1 {
    // addition
} else if choice == 2 {
    // subtraction
}
```

you can use:

```rust
match choice {
    1 => println!("Addition"),
    2 => println!("Subtraction"),
    3 => println!("Multiplication"),
    4 => println!("Division"),
    5 => println!("Exit"),
    _ => println!("Invalid choice"),
}
```

You can make it perform the operations:

```rust
match choice {
    1 => println!("Result: {}", a + b),
    2 => println!("Result: {}", a - b),
    3 => println!("Result: {}", a * b),
    4 => {
        if b == 0.0 {
            println!("Cannot divide by zero");
        } else {
            println!("Result: {}", a / b);
        }
    }
    5 => println!("Goodbye!"),
    _ => println!("Invalid choice"),
}
```

This is a perfect exercise for learning `match`.

---

# 15. Backend Connection

Eventually you'll see things like:

```rust
match result {
    Ok(user) => {
        // return user
    }
    Err(error) => {
        // return error response
    }
}
```

Or:

```rust
match user {
    Some(user) => {
        // user exists
    }
    None => {
        // user doesn't exist
    }
}
```

Or:

```rust
match request.method() {
    Method::GET => {
        // handle GET
    }
    Method::POST => {
        // handle POST
    }
    _ => {
        // other methods
    }
}
```

So `match` isn't just a Rust syntax feature you'll learn and forget. **It's one of the mechanisms you'll use to handle states and outcomes throughout your backend applications.**

---

# What you should know before moving on

Based on the roadmap, get comfortable with:

- Definition of `match`.
- Understand `pattern => expression`.
- Use `_` as the catch-all.
- Understand why `match` must be exhaustive.
- Return values from `match`.
- Match enums.
- Extract data from enum variants.
- Match `Option<T>`.
- Match `Result<T, E>`.
- Understand basic match guards.
- Know when `match` is more appropriate than `if`.

### Most important patterns to remember

```rust
match value {
    1 => println!("One"),
    2 => println!("Two"),
    _ => println!("Other"),
}
```

```rust
match option {
    Some(value) => println!("{}", value),
    None => println!("Nothing"),
}
```

```rust
match result {
    Ok(value) => println!("{}", value),
    Err(error) => println!("{}", error),
}
```
