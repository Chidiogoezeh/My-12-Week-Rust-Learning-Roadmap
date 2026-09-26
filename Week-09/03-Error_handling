# Error Handling in Rust

For a Rust backend, you should understand **how errors are created, returned, propagated, converted, and turned into HTTP responses**.

The goal is not to make every error system complicated. For now, focus on these concepts:

1. `Result<T, E>`
2. `Error` types
3. `?` operator
4. `match` and `map_err`
5. Custom application errors
6. Converting errors with `From`
7. Returning proper HTTP status codes
8. Logging errors with `tracing`

---

## 1. `Result<T, E>`

Rust uses `Result` for operations that can succeed or fail.

```rust
fn divide(a: i32, b: i32) -> Result<i32, String> {
    if b == 0 {
        return Err("Cannot divide by zero".to_string());
    }

    Ok(a / b)
}
```

`Result` has two possibilities:

```rust
Ok(value)
Err(error)
```

Using it:

```rust
match divide(10, 2) {
    Ok(result) => println!("Result: {}", result),
    Err(error) => println!("Error: {}", error),
}
```

Output:

```text
Result: 5
```

---

# 2. Why `Result` matters in a backend

Backend operations fail all the time:

- Database query fails
- User doesn't exist
- Invalid input
- Authentication fails
- File cannot be read
- External API fails

Instead of hiding those failures, Rust forces you to deal with them.

Example:

```rust
async fn find_user() -> Result<User, String> {
    // database operation
}
```

This tells the caller:

> This function can return either a `User` or an error.

---

# 3. The `?` operator

The `?` operator is one of the most important things you need to understand.

Instead of:

```rust
fn get_user() -> Result<User, String> {
    let user = match find_user() {
        Ok(user) => user,
        Err(error) => return Err(error),
    };

    Ok(user)
}
```

You can write:

```rust
fn get_user() -> Result<User, String> {
    let user = find_user()?;

    Ok(user)
}
```

`?` means:

> If this succeeds, give me the value. If it fails, return the error from this function.

Another example:

```rust
fn create_user() -> Result<User, String> {
    let data = validate_user()?;
    let user = save_user(data)?;

    Ok(user)
}
```

The flow is:

```text
validate_user()
      |
      | success
      v
save_user()
      |
      | success
      v
Ok(user)

If either fails:
      |
      v
return Err(...)
```

You will use `?` constantly in backend Rust.

---

# 4. Functions that use `?` must return `Result`

This will not work:

```rust
fn get_user() -> User {
    let user = find_user()?;

    user
}
```

Because `?` needs the function to be able to return an error.

Correct:

```rust
fn get_user() -> Result<User, String> {
    let user = find_user()?;

    Ok(user)
}
```

Remember:

```text
? → function must return Result or another compatible type
```

---

# 5. `match` for handling errors

You should still understand `match`.

```rust
match find_user() {
    Ok(user) => {
        println!("User: {:?}", user);
    }

    Err(error) => {
        println!("Error: {}", error);
    }
}
```

Use `match` when you need to do something different depending on the result.

Use `?` when you simply want to propagate the error.

---

# 6. Custom errors

Using `String` for every error is not good for a real backend.

For example:

```rust
Result<User, String>
```

becomes difficult to manage when your application has many error types.

Instead, create an application error enum.

```rust
#[derive(Debug)]
enum AppError {
    NotFound,
    DatabaseError,
    ValidationError,
    Unauthorized,
}
```

Now:

```rust
fn find_user(id: i32) -> Result<User, AppError> {
    if id == 0 {
        return Err(AppError::NotFound);
    }

    // ...
}
```

This gives your application meaningful error categories.

---

# 7. Matching custom errors

You can handle them with `match`:

```rust
match find_user(10) {
    Ok(user) => println!("User: {:?}", user),

    Err(AppError::NotFound) => {
        println!("User not found");
    }

    Err(AppError::DatabaseError) => {
        println!("Database error");
    }

    Err(error) => {
        println!("Other error: {:?}", error);
    }
}
```

---

# 8. Converting errors with `From`

This becomes important when working with libraries.

For example, SQLx can return a `sqlx::Error`.

Your application has its own `AppError`.

You can implement:

```rust
impl From<sqlx::Error> for AppError {
    fn from(_: sqlx::Error) -> Self {
        AppError::DatabaseError
    }
}
```

Now this works:

```rust
async fn find_user(pool: &PgPool) -> Result<User, AppError> {
    let user = sqlx::query_as!(
        User,
        "SELECT id, username FROM users WHERE id = $1",
        1
    )
    .fetch_one(pool)
    .await?;

    Ok(user)
}
```

The important part is:

```rust
.await?
```

The SQLx error is automatically converted into your `AppError` through `From`.

Mental model:

```text
sqlx::Error
     |
     | From
     v
AppError
```

---

# 9. `map_err`

Sometimes you want to manually convert an error.

```rust
let user = find_user()
    .map_err(|_| AppError::NotFound)?;
```

`map_err` means:

> Convert this error into another error.

Example:

```rust
let user = database_call()
    .map_err(|error| {
        tracing::error!(error = ?error, "Database query failed");
        AppError::DatabaseError
    })?;
```

You don't need to use `map_err` everywhere. Use it when you need custom conversion or additional handling.

---

# 10. Errors in Axum

Your backend should turn internal errors into appropriate HTTP responses.

For example:

```text
User not found       → 404
Invalid input        → 400
Not authenticated    → 401
Forbidden            → 403
Database failure     → 500
```

Create an error type:

```rust
use axum::{
    http::StatusCode,
    response::{IntoResponse, Response},
};

enum AppError {
    NotFound,
    ValidationError,
    DatabaseError,
    Unauthorized,
}
```

Then implement `IntoResponse`:

```rust
impl IntoResponse for AppError {
    fn into_response(self) -> Response {
        let (status, message) = match self {
            AppError::NotFound => {
                (StatusCode::NOT_FOUND, "User not found")
            }

            AppError::ValidationError => {
                (StatusCode::BAD_REQUEST, "Invalid input")
            }

            AppError::DatabaseError => {
                (StatusCode::INTERNAL_SERVER_ERROR, "Internal server error")
            }

            AppError::Unauthorized => {
                (StatusCode::UNAUTHORIZED, "Unauthorized")
            }
        };

        (status, message).into_response()
    }
}
```

Now your handler can simply return:

```rust
async fn get_user() -> Result<String, AppError> {
    let user = find_user()?;

    Ok(user.username)
}
```

Axum sees the `AppError` and uses `IntoResponse` to produce the HTTP response.

---

# 11. The important backend flow

This is the pattern you should understand:

```text
HTTP Request
     |
     v
Handler
     |
     v
Service
     |
     v
Repository
     |
     v
Database
```

Suppose the database fails:

```text
Database
   |
   | sqlx::Error
   v
Repository
   |
   | ?
   v
Service
   |
   | ?
   v
Handler
   |
   | AppError
   v
HTTP 500
```

You don't need to handle the same error repeatedly at every layer.

Use `?` to propagate it upward and handle it at the appropriate boundary.

---

# 12. Don't expose internal errors to users

Bad:

```rust
return Err(format!("Database error: {}", error));
```

You generally don't want users seeing:

```text
Database error: connection refused to PostgreSQL at 192.168...
```

Instead:

```text
Internal server error
```

But log the real error internally:

```rust
tracing::error!(
    error = ?error,
    "Database query failed"
);
```

So:

```text
Client:
500 Internal Server Error

Server logs:
Database query failed: connection refused...
```

This is an important production practice.

---

# 13. Error handling + validation

This connects directly to what you learned earlier.

Example:

```rust
async fn register(
    Json(payload): Json<RegisterRequest>,
) -> Result<String, AppError> {

    payload
        .validate()
        .map_err(|_| AppError::ValidationError)?;

    create_user(payload).await?;

    Ok("User created".to_string())
}
```

Flow:

```text
Request
   |
   v
Validation
   |
   | invalid
   v
400 Bad Request

   |
   | valid
   v
Business logic
   |
   v
Database
   |
   | failure
   v
500 Internal Server Error
```

---

# 14. What you should NOT do

Avoid this style:

```rust
unwrap()
```

and:

```rust
expect("something went wrong")
```

in normal request handling.

For example:

```rust
let user = find_user().unwrap();
```

If the operation fails, your application can panic.

Prefer:

```rust
let user = find_user()?;
```

Use `unwrap()` mainly when you genuinely know something cannot fail, particularly in controlled test/setup code.

---

# 15. `panic!` vs `Result`

Know this distinction:

### Expected failure

Use:

```rust
Result<T, E>
```

Examples:

```text
Invalid login
User not found
Database unavailable
Invalid request
```

### Unexpected programming failure

Can involve:

```rust
panic!
```

Examples:

```text
An impossible internal state
A violated invariant
A programming bug
```

For normal backend request failures, think:

```text
Result, not panic.
```

---

# Error Handling Checklist

By the end of the week, you should be able to explain and use:

- `Result<T, E>`
- `Ok`
- `Err`
- `match`
- `?`
- Error propagation
- Custom error enums
- `From`
- `map_err`
- Axum `IntoResponse`
- HTTP error status codes
- Validation errors → `400`
- Authentication errors → `401`
- Authorization errors → `403`
- Not found → `404`
- Internal/database errors → `500`
- Logging internal errors with `tracing`
- Avoiding `unwrap()` in request handling
- Not exposing internal errors to clients

### The most important pattern to remember

```rust
async fn handler() -> Result<Response, AppError> {
    validate()?;
    let user = find_user().await?;
    save_user(&user).await?;

    Ok(response)
}
```

Think of `?` as:

> **"If this operation fails, stop here and pass the error upward."**

That is the core of Rust backend error handling.
