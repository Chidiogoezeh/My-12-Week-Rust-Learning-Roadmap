`Result` is one of the most important Rust concepts to understand because it is used constantly in backend development.

The core idea is simple:

> **`Result` represents an operation that can either succeed or fail.**

# 1. The two possible outcomes

`Result<T, E>` has two variants:

```rust
Ok(value)
Err(error)
```

Think:

```text
Result<T, E>
    │
    ├── Ok(T)  → operation succeeded
    │
    └── Err(E) → operation failed
```

For example:

```rust
let result: Result<i32, String> = Ok(100);
```

means:

> The operation succeeded and returned `100`.

While:

```rust
let result: Result<i32, String> =
    Err(String::from("Something went wrong"));
```

means:

> The operation failed and returned an error message.

---

# 2. Why does Rust use `Result`?

Suppose you divide two numbers:

```rust
10 / 2
```

No problem.

But:

```rust
10 / 0
```

is an error.

Instead of silently returning some special value, Rust can make the possibility of failure explicit:

```rust
Result<i32, String>
```

The caller must decide what to do with the success or failure.

---

# 3. Creating a `Result`

A successful result:

```rust
let result: Result<i32, String> = Ok(100);
```

An error:

```rust
let result: Result<i32, String> =
    Err(String::from("Something went wrong"));
```

---

# 4. Using `match`

This is the most important way to understand `Result`.

```rust
let result: Result<i32, String> = Ok(100);

match result {
    Ok(value) => println!("Success: {}", value),
    Err(error) => println!("Error: {}", error),
}
```

Output:

```text
Success: 100
```

If the result is:

```rust
let result: Result<i32, String> =
    Err(String::from("Something went wrong"));
```

you get:

```text
Error: Something went wrong
```

The pattern is:

```rust
match result {
    Ok(value) => {
        // success
    }

    Err(error) => {
        // failure
    }
}
```

You should be very comfortable reading this.

---

# 5. Returning `Result` from a function

This is where `Result` becomes really useful.

Let's create a division function:

```rust
fn divide(a: i32, b: i32) -> Result<i32, String> {
    if b == 0 {
        Err(String::from("Cannot divide by zero"))
    } else {
        Ok(a / b)
    }
}
```

The return type:

```rust
Result<i32, String>
```

means:

```text
Success → i32
Failure → String
```

Use it:

```rust
fn main() {
    let result = divide(10, 2);

    match result {
        Ok(value) => println!("Result: {}", value),
        Err(error) => println!("Error: {}", error),
    }
}
```

Output:

```text
Result: 5
```

Try:

```rust
let result = divide(10, 0);
```

Output:

```text
Error: Cannot divide by zero
```

---

# 6. `Ok` and `Err`

Think of them like this:

```text
divide(10, 2)
       ↓
    Ok(5)


divide(10, 0)
       ↓
Err("Cannot divide by zero")
```

This is the fundamental idea behind `Result`.

---

# 7. `if let`

If you only care about success:

```rust
let result = divide(10, 2);

if let Ok(value) = result {
    println!("Result: {}", value);
}
```

But if you need to handle both success and failure, `match` is clearer:

```rust
match divide(10, 2) {
    Ok(value) => println!("Result: {}", value),
    Err(error) => println!("Error: {}", error),
}
```

---

# 8. `unwrap()`

You will see this frequently:

```rust
let result = divide(10, 2);

let value = result.unwrap();

println!("{}", value);
```

Output:

```text
5
```

`unwrap()` extracts the value from `Ok`.

But if the result is:

```rust
let result = divide(10, 0);
```

then:

```rust
result.unwrap();
```

will **panic**.

So don't use `unwrap()` blindly.

It's fine for simple learning examples and situations where failure is genuinely impossible or deliberately treated as a programmer error, but production backend code generally handles errors explicitly.

---

# 9. `unwrap_or()`

You can provide a fallback value:

```rust
let result = divide(10, 0);

let value = result.unwrap_or(0);

println!("{}", value);
```

Output:

```text
0
```

If successful:

```rust
let result = divide(10, 2);

let value = result.unwrap_or(0);

println!("{}", value);
```

Output:

```text
5
```

So:

```text
Ok(5)  → 5
Err(_) → 0
```

---

# 10. The `?` operator — VERY important

This is one of the most important things you'll eventually use in backend Rust.

Suppose:

```rust
fn divide(a: i32, b: i32) -> Result<i32, String> {
    if b == 0 {
        Err(String::from("Cannot divide by zero"))
    } else {
        Ok(a / b)
    }
}
```

Now another function needs to use it:

```rust
fn calculate() -> Result<i32, String> {
    let result = divide(10, 2)?;

    Ok(result + 10)
}
```

The `?` means roughly:

> If this is `Ok`, give me the value. If this is `Err`, immediately return the error from this function.

So:

```rust
let result = divide(10, 2)?;
```

is conceptually similar to:

```rust
let result = match divide(10, 2) {
    Ok(value) => value,
    Err(error) => return Err(error),
};
```

You don't need to memorize that expanded version yet. Just understand what `?` does.

### Important requirement

A function using `?` must return a compatible `Result` (or another compatible type supporting `Try`).

For example:

```rust
fn calculate() -> Result<i32, String>
```

---

# 11. Practical example with `?`

```rust
fn divide(a: i32, b: i32) -> Result<i32, String> {
    if b == 0 {
        return Err(String::from("Cannot divide by zero"));
    }

    Ok(a / b)
}

fn calculate() -> Result<i32, String> {
    let result = divide(10, 2)?;

    Ok(result + 5)
}

fn main() {
    match calculate() {
        Ok(value) => println!("Result: {}", value),
        Err(error) => println!("Error: {}", error),
    }
}
```

Output:

```text
Result: 10
```

If `divide()` fails, the error automatically travels back through `calculate()`.

This becomes extremely useful when you have several operations that can fail.

---

# 12. `Result` with file operations

Here's a more realistic example.

Reading a file can fail because:

- The file doesn't exist.
- You don't have permission.
- The path is wrong.

Rust therefore returns a `Result`.

```rust
use std::fs;

fn main() {
    let result = fs::read_to_string("data.txt");

    match result {
        Ok(contents) => println!("{}", contents),
        Err(error) => println!("Could not read file: {}", error),
    }
}
```

The important part is:

```rust
fs::read_to_string("data.txt")
```

returns a `Result`.

So you have to account for both:

```text
Ok(contents)
```

and:

```text
Err(error)
```

---

# 13. `Result` in backend development

This is where `Result` becomes extremely important for your roadmap.

Imagine your API needs to create a user:

```text
HTTP request
     ↓
Validate data
     ↓
Hash password
     ↓
Insert into PostgreSQL
     ↓
Return response
```

Several things can fail:

```text
Validation       → Err
Password hashing → Err
Database query   → Err
```

Instead of hiding these errors, Rust lets functions return:

```rust
Result<User, Error>
```

Conceptually:

```text
              create_user()
                    │
            ┌───────┴────────┐
            ↓                ↓
         Ok(User)          Err(Error)
            ↓                ↓
       HTTP 201            HTTP 400/500
```

You'll use this pattern constantly with Axum and SQLx.

---

# 14. `Result` vs `Option`

This distinction is **essential**.

### `Option`

Something **may or may not exist**:

```rust
Option<User>
```

Means:

```text
Some(User) → user exists
None       → user doesn't exist
```

### `Result`

An operation **succeeded or failed**:

```rust
Result<User, Error>
```

Means:

```text
Ok(User) → operation succeeded
Err(e)   → operation failed
```

Think:

```text
Option
"Is there a value?"

Some / None


Result
"Did the operation succeed?"

Ok / Err
```

For example:

```rust
find_user(id)
```

might reasonably return:

```rust
Option<User>
```

because the user might simply not exist.

But:

```rust
create_user(data)
```

might return:

```rust
Result<User, Error>
```

because creating the user involves operations that can fail.

---

# 15. The methods/syntax you need now

For result, focus on:

| Concept            | Purpose                       |
| ------------------ | ----------------------------- |
| `Ok(value)`        | Successful result             |
| `Err(error)`       | Failed result                 |
| `match`            | Handle success/failure        |
| `if let Ok(value)` | Handle success                |
| `unwrap()`         | Extract value, panic on error |
| `unwrap_or()`      | Extract value or use fallback |
| `?`                | Propagate an error            |

You **do not need** to memorize every `Result` method yet.

---

# Topic target

You should be able to write and understand this:

```rust
fn divide(a: i32, b: i32) -> Result<i32, String> {
    if b == 0 {
        Err(String::from("Cannot divide by zero"))
    } else {
        Ok(a / b)
    }
}

fn main() {
    match divide(10, 2) {
        Ok(value) => println!("Result: {}", value),
        Err(error) => println!("Error: {}", error),
    }
}
```

And you should be able to explain:

```rust
Result<i32, String>
```

as:

> This function either returns an `i32` if successful or a `String` describing the error if it fails.

Then understand:

```text
Ok(5)
```

means success, while:

```text
Err("Cannot divide by zero")
```

means failure.

Finally, understand this:

```rust
let value = divide(10, 2)?;
```

as:

> Give me the successful value; if there's an error, stop here and return that error.

### Topic mental model

```text
             Result<T, E>
                  │
          ┌───────┴───────┐
          ↓               ↓
       Ok(T)            Err(E)
          │               │
      succeeded          failed
          │               │
       value             error
```

If you understand **`Ok`, `Err`, `match`, returning `Result` from functions, `unwrap`, `unwrap_or`, and especially the `?` operator**, you know enough `Result`.
