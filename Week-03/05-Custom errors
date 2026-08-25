There is a need to understand custom errors well enough to stop returning random `String` errors everywhere and start creating meaningful error types.

The core idea:

> A **custom error** is an error type you create yourself to describe the specific things that can go wrong in your application.

---

# 1. Why custom errors?

You might start with:

```rust
fn divide(a: i32, b: i32) -> Result<i32, String> {
    if b == 0 {
        Err(String::from("Cannot divide by zero"))
    } else {
        Ok(a / b)
    }
}
```

This works.

But `String` doesn't tell us **what kind of error** occurred.

As your application grows, you might have:

```text
"User not found"
"Invalid email"
"Database error"
"Permission denied"
```

Using strings for everything becomes difficult to manage.

Instead, create an error type.

---

# 2. Create a custom error with `enum`

This is the most important approach to learn now.

```rust
enum AppError {
    UserNotFound,
    InvalidEmail,
    Unauthorized,
}
```

You've created your own error type:

```text
AppError
```

with three possible errors:

```text
UserNotFound
InvalidEmail
Unauthorized
```

---

# 3. Use it with `Result`

Now a function can return:

```rust
Result<T, AppError>
```

Example:

```rust
enum AppError {
    UserNotFound,
}

fn find_user(id: u32) -> Result<String, AppError> {
    if id == 1 {
        Ok(String::from("John"))
    } else {
        Err(AppError::UserNotFound)
    }
}
```

Here:

```rust
Result<String, AppError>
```

means:

```text
Success → String
Failure → AppError
```

---

# 4. Handle the custom error with `match`

```rust
fn main() {
    match find_user(10) {
        Ok(name) => println!("User: {}", name),
        Err(AppError::UserNotFound) => {
            println!("User not found");
        }
    }
}
```

Output:

```text
User not found
```

This is the basic custom-error pattern you should understand.

---

# 5. Multiple error types

You can have multiple variants:

```rust
enum AppError {
    UserNotFound,
    InvalidEmail,
    Unauthorized,
}
```

Then:

```rust
fn check_user(id: u32) -> Result<String, AppError> {
    if id == 0 {
        Err(AppError::UserNotFound)
    } else {
        Ok(String::from("John"))
    }
}
```

And handle them:

```rust
match check_user(0) {
    Ok(name) => println!("User: {}", name),

    Err(AppError::UserNotFound) => {
        println!("User not found");
    }

    Err(AppError::InvalidEmail) => {
        println!("Invalid email");
    }

    Err(AppError::Unauthorized) => {
        println!("Unauthorized");
    }
}
```

However, Rust will tell you if a `match` doesn't cover all possible enum variants.

---

# 6. Custom errors can contain information

This is very useful.

Instead of:

```rust
enum AppError {
    UserNotFound,
}
```

you can have:

```rust
enum AppError {
    UserNotFound(u32),
}
```

The `u32` can contain the user ID.

Example:

```rust
fn find_user(id: u32) -> Result<String, AppError> {
    if id == 1 {
        Ok(String::from("John"))
    } else {
        Err(AppError::UserNotFound(id))
    }
}
```

Then:

```rust
match find_user(10) {
    Ok(name) => println!("User: {}", name),

    Err(AppError::UserNotFound(id)) => {
        println!("User {} was not found", id);
    }
}
```

Output:

```text
User 10 was not found
```

---

# 7. Custom errors with structs

You can also create a struct error:

```rust
struct AppError {
    message: String,
}
```

Then:

```rust
fn do_something() -> Result<(), AppError> {
    Err(AppError {
        message: String::from("Something went wrong"),
    })
}
```

And:

```rust
match do_something() {
    Ok(_) => println!("Success"),

    Err(error) => println!("{}", error.message),
}
```

For your current level, however, **focus primarily on enum-based errors**.

They're particularly useful because your application can have different categories of errors.

---

# 8. `Debug` — you'll need this

You'll often want to print an error with `{:?}`.

Add:

```rust
#[derive(Debug)]
```

Example:

```rust
#[derive(Debug)]
enum AppError {
    UserNotFound,
    Unauthorized,
}
```

Now:

```rust
fn main() {
    let error = AppError::UserNotFound;

    println!("{:?}", error);
}
```

Output:

```text
UserNotFound
```

`Debug` is mainly for developer/debug output.

---

# 9. `Display` — for human-readable errors

Eventually you'll want:

```rust
println!("{}", error);
```

instead of:

```rust
println!("{:?}", error);
```

For that, Rust uses the `Display` trait.

Example:

```rust
use std::fmt;

#[derive(Debug)]
enum AppError {
    UserNotFound,
    Unauthorized,
}

impl fmt::Display for AppError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            AppError::UserNotFound => {
                write!(f, "User not found")
            }

            AppError::Unauthorized => {
                write!(f, "Unauthorized")
            }
        }
    }
}
```

Then:

```rust
fn main() {
    let error = AppError::UserNotFound;

    println!("{}", error);
}
```

Output:

```text
User not found
```

### What should you understand now?

You don't need to memorize the `fmt` implementation yet.

Just understand:

```text
Debug
{:?}
↓
developer/debug output

Display
{}
↓
human-readable output
```

---

# 10. Custom errors + `?`

This is where custom errors become powerful.

```rust
#[derive(Debug)]
enum AppError {
    UserNotFound,
}

fn find_user(id: u32) -> Result<String, AppError> {
    if id == 1 {
        Ok(String::from("John"))
    } else {
        Err(AppError::UserNotFound)
    }
}

fn get_username(id: u32) -> Result<String, AppError> {
    let name = find_user(id)?;

    Ok(name)
}
```

The important part:

```rust
let name = find_user(id)?;
```

If `find_user()` returns:

```rust
Ok(name)
```

the name continues.

If it returns:

```rust
Err(AppError::UserNotFound)
```

the error is returned from `get_username()`.

---

# 11. A simple backend-style example

Imagine your backend has users.

```rust
#[derive(Debug)]
enum AppError {
    UserNotFound,
    InvalidEmail,
}

fn find_user(email: &str) -> Result<String, AppError> {
    if email == "john@example.com" {
        Ok(String::from("John"))
    } else {
        Err(AppError::UserNotFound)
    }
}
```

Then:

```rust
fn main() {
    match find_user("mary@example.com") {
        Ok(name) => println!("Welcome {}", name),

        Err(AppError::UserNotFound) => {
            println!("User not found");
        }

        Err(AppError::InvalidEmail) => {
            println!("Invalid email");
        }
    }
}
```

This is conceptually similar to what you'll eventually build with Axum:

```text
Request
   ↓
Handler
   ↓
Service
   ↓
Database
   ↓
Result<T, AppError>
   ↓
┌──────────────┐
│              │
Ok            Err
│              │
Success       Error
response      response
```

---

# 12. Why custom errors matter for your backend

Later you'll have errors such as:

```text
UserNotFound
InvalidCredentials
ValidationError
DatabaseError
Unauthorized
Forbidden
NotFound
```

Instead of having functions return:

```rust
Result<User, String>
```

you can eventually have:

```rust
Result<User, AppError>
```

This makes your application much easier to organize.

---

# What you need to know for starters

Focus on these concepts:

| Concept            | What you should understand    |
| ------------------ | ----------------------------- |
| `enum` error       | Create your own error type    |
| `Result<T, E>`     | Return your custom error      |
| Error variants     | Different types of failures   |
| Data in variants   | Attach useful information     |
| `match`            | Handle different errors       |
| `#[derive(Debug)]` | Print/debug the error         |
| `Display`          | Human-readable error messages |
| `?`                | Propagate custom errors       |

### Don't worry about these yet

You **do not** need to learn:

- `thiserror`
- `anyhow`
- Complex error conversions
- Error source chains
- Backtraces
- Advanced generic error types

Those become useful later when your projects get bigger.

---

# Topic target

You should be able to understand and write something like this:

```rust
#[derive(Debug)]
enum AppError {
    UserNotFound,
    InvalidEmail,
}

fn find_user(id: u32) -> Result<String, AppError> {
    if id == 1 {
        Ok(String::from("John"))
    } else {
        Err(AppError::UserNotFound)
    }
}

fn main() {
    match find_user(10) {
        Ok(name) => println!("User: {}", name),

        Err(AppError::UserNotFound) => {
            println!("User not found");
        }

        Err(AppError::InvalidEmail) => {
            println!("Invalid email");
        }
    }
}
```

The key mental model is:

```text
Custom error
     ↓
enum AppError
     ↓
┌──────────────┬────────────────┐
UserNotFound   InvalidEmail    ...
     ↓               ↓
     └────── Result<T, AppError>
                     ↓
               Ok(...) / Err(...)
```

If you understand **why you'd create `AppError` instead of using `String`, how enum variants represent different failures, how to return them through `Result`, and how `?` propagates them**, you've learned enough custom errors for starters.
